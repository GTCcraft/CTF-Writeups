# You Work For Me Now: pwn/tic-tac-no 
> Tic-tac-toe is a draw when played perfectly. Can you be more perfect than my perfect bot?

This pwn challenge presents a command line game of tic-tac-toe, to be played by the user against a bot that generates perfect countermoves. We take advantage of a flawed array bounds check to override the computer's piece with our own piece, making the computer play for us and thus winning the game with the aid of our adversary. 

## Initial impression

According to the challenge description, we can run the game with `nc chall.lac.tf 30001`:

<img width="1123" height="1006" alt="Screenshot 2026-02-16 232402" src="https://github.com/user-attachments/assets/847600d1-10fb-46ca-a3da-2ff7e9aac802" />

We're able to input the row and column at which to place our piece. Notably, the game doesn't complain when our index is far outside the board; it just keeps playing. I play a few rounds and lose, obviously. 

We're given the code for the tic-tac-toe program in a file called chall.c. Inside main(), we see the loop where the game takes place. After the loop breaks, we see the check that outputs the flag:
```c
 if (winner == player) {
      ... Prints the flag.
   }
   else {
      printf("Nice try, but I'm still unbeatable.\n");
   }
```

Further down the file, we see the code for playerMove(), which contains an incriminating comment:

```c
void playerMove() {
   ...
      if(index >= 0 && index < 9 && board[index] != ' '){
         printf("Invalid move.\n");
      }else{
         board[index] = player; // Should be safe, given that the user cannot overwrite tiles on the board
         break;
      }
   }while(1);
}
```

Should be safe? 

## How do we use board[index]? 

My first instinct is to overflow the integer index, which is calculated via `int index = (x-1)*3+(y-1);` --- make it a huge number that bypasses the check `if(index >= 0 && index < 9 && board[index] != ' ')`, but that somehow registers as a small index in `board[index] = player;`. However, I realized this would be functionally identical to passing a small number as index. 

At this point, I ask Gemini how to do pwn challenges. It tells me to examine the compiled chall binary (which we're given with the challenge) using pwndbg. Sounds good. 

I download pwndbg and spend 15 minutes poking around. I try to view the source code alongside the assembly and I realze chall doesn't come with a symbol table...? Fine, I'll do it myself. 

```
> gcc -g chall.c -o yeah
> pwndbg yeah
```

The gcc flag "-g" causes the executable to be compiled with a symbol table, which lets pwndbg associate lines of source code with chunks of assembly. Apparently. 

Anyway, I give Gemini the source code and ask how to get started with pwndbg. It says: 
    
-   **The Goal:** You don't actually need to "win" the game of Tic-Tac-Toe. You just need the `winner` variable to contain the character `'X'` (the value of `player`) when the loop ends.

So I need to set `winner` to 'X' using `board[index] = player;`? 

I then run `pwndbg > distance &board &winner` to find how far `winner` is from `board` in memory. This gives me an absurdly large number in hex. I convert it to a decimal number; let's call it N. 

Recall that index = (x-1)*3+(y-1). I attempt to input x = (N / 3) + 1, y = (N % 3) + 1, but this just makes pwndbg complain that the memory address can't be dereferenced. 

Maybe this is a dead end. I go complain to Gemini, which says:
If you can't reach the stack to change `winner`, could you change `player` or `computer` to something else to trick the `checkWin()` logic? Or is there another way to make `winner == player`?

I refer back to the game loop in main() and note these lines:
```c
playerMove();
winner  =  checkWin();
```
So actually, even if I set winner to 'X' during playerMove(), it's immediately set to the true winner in checkWin(), and that winner is surely not me. 

## Hijacking the mainframe 
I ponder my orb for a moment. The only thing I can do here is put an 'X' somewhere in memory, but where?

Wait a sec:
```c
char board[9];
char player = 'X';
char computer = 'O';
```

`computer = 'O'` you say? Not for long!

```
pwndbg> distance &board &computer
0x555555558028->0x555555558011 is -0x17 bytes (-0x3 words)
```

Can I use a negative index in an array? Surely. C doesn't even bother to check array bounds, and anyway, it's all just arithmetic in there. 

Sure enough, I ask Gemini and it says:
In C, an array index is just a pointer offset. When you write `board[index]`, the computer calculates the memory address as `board + index`. If `index` is negative, it simply looks at memory addresses **before** the array starts.

Sounds good. 0x17 = 23, so I need to write 'X' to board[-23]. 

Given `index=(x−1)×3+(y−1)`, we need x = -6 and y = -1 to produce -23. 

Sure enough, I set a breakpoint before the computer move and input my x and y:
<img width="1657" height="908" alt="Screenshot 2026-02-16 233236" src="https://github.com/user-attachments/assets/59fc20a3-e3fb-48d6-a9e1-0bc81499dedc" />

And when I check the value of `computer`:
<img width="939" height="197" alt="Screenshot 2026-02-16 233327" src="https://github.com/user-attachments/assets/389fa0a0-3172-4587-b017-aa230fe94e6b" />

I allow the game to continue, and the computer plays my piece:
<img width="562" height="723" alt="Screenshot 2026-02-16 233508" src="https://github.com/user-attachments/assets/a8e2fa34-4f7c-4aa3-8fcb-371a2cd4d08e" />

Now I simply play x = 1, y = 1, and the computer, in its futile and impotent effort to prevent my inexorable victory, instead seals its own doom:
<img width="1964" height="729" alt="Screenshot 2026-02-16 233727" src="https://github.com/user-attachments/assets/c9a5c916-23e0-465e-bf01-7cd20100a84d" />

...Then it segfaults. Maybe because it's my own version of chall, and it's trying to find a flag that doesn't exist. Anyway:
<img width="1824" height="1364" alt="Screenshot 2026-02-16 233826" src="https://github.com/user-attachments/assets/ae3a4c32-717d-419e-9c87-d361acc8f985" />

Gg, my wretched nemesis. 

Flag: lactf{th3_0nly_w1nn1ng_m0ve_1s_t0_p1ay}
