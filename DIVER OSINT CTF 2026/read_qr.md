# A Matter of Perspective: read_qr 
> What URL is stored in this QR code?
<img width="2048" height="1542" alt="image" src="https://github.com/user-attachments/assets/f2a5e24a-56c6-47f9-9492-dad029db4b1e" />

Simply scanning the QR code with a phone camera will strip off the end of the URL and give us the wrong answer. We can use CyberChef to parse the QR code and reveal the true contents, but first we have to manipulate the photo using Photopea's perspective tools so that CyberChef can actually read it.
## Initial impression
"Uh," I said, vaguely to my teammates, mostly to myself, "Can I just... scan it?"

I could. It took me here:

<img width="796" height="1424" alt="image" src="https://github.com/user-attachments/assets/5cd5aa17-6a43-4039-bc95-75f0a1bf53ba" />

Well, it couldn't be that easy. It was at this point that I checked the CTF's end time and realized I had 14 minutes to solve this challenge. Suddenly feeling like that guy from the high cortisol meme, I fired off this ham-fisted prompt to Gemini:
> For this OSINT challenge, I'm asked to answer "What URL is stored in this QR code?" How are URLs encoded into QR codes/what is really meant by "a URL being stored in a QR code"? When I scan it, I'm taken to this website, which I guess is a dead end? Where do i go from here 

I learn that QR codes are ultimately just a visual way to encode text. I'm also told:
> Smartphones are designed for user convenience, not data integrity. When your phone's built-in camera decodes a string that looks like a web address, it often normalizes the text before passing it to the browser […]
> The Cloudflare worker at qr.intro26.workers.dev is explicitly telling you that the normalized, lowercase URL your phone resolved is the wrong answer. The challenge asks for the URL stored in the code, which means you need the literal, unmodified string embedded in that grid.

Gemini then directs me to Zebra Crossing’s barcode processing site:
<img width="2048" height="849" alt="image" src="https://github.com/user-attachments/assets/3741e8f5-d725-48c8-9dbc-7bc0dfb44748" />

I upload the image of the QR code, and...
<img width="1678" height="318" alt="image" src="https://github.com/user-attachments/assets/d9b54570-7069-4e42-b9f2-f8c1bcba8956" />

Hmm. Not ideal. I turn to Gemini's second recommendation, CyberChef and its "Parse QR Code" function. In goes the image...
<img width="1232" height="980" alt="image" src="https://github.com/user-attachments/assets/bf30f942-5ab1-4482-96bb-ed29d46ee6f8" />

Well, alright. I begin to suspect the perspective skew in the picture is tripping up these QR code processors. With 8 minutes to go, I draw on my illustrious history as a digital artist and remember Photopea's perspective transform tool. (Photopea, for those lacking ball knowledge, is basically Photoshop but free in your browser, ideal for slopping together stupid recreations of scenes from your ongoing D&D campaign. Or, you know, various other image editing applications.) I drop the image in:

<img width="2292" height="1474" alt="image" src="https://github.com/user-attachments/assets/df4aba6d-3016-4eba-82f8-50b68aad3395" />

And then run to Google for instructions:
<img width="1420" height="540" alt="image" src="https://github.com/user-attachments/assets/d42f7e9a-7c87-4020-b8b9-355c4496c272" />

I first select a square containing the QR code, then invert the selection to delete everything else in the picture. Then I do as instructed above, dragging the transform handles around until I get... something. 

<img width="406" height="610" alt="image" src="https://github.com/user-attachments/assets/80dd408d-bf3c-4dbf-a147-897c0a4e03a9" />

I'm not altogether convinced that CyberChef will accept this thing, freakishly tall and slanted, the corners shaved into transparency. But I load it in, and lo:
<img width="1418" height="1058" alt="image" src="https://github.com/user-attachments/assets/af68aac5-ee22-458c-9e51-b07a745d4988" />

The URL is revealed. I grab it for the flag and plug it into the challenge, which flashes green acceptance. Only then do I double back and read the results of the prompt I'd hammered out to Gemini in a panic:
> neither cyberchef nor the other website is able to detect the QR code. should I edit the image somehow 

Gemini:
> Use the "Perspective Crop" tool (usually hidden under the standard Crop tool on the left toolbar). Click the four corners of the QR code itself, and hit Enter. This will mathematically flatten the image out into a perfect square, completely removing the angle. 

<img width="1506" height="980" alt="image" src="https://github.com/user-attachments/assets/37b48c3e-338e-4c19-aa6b-04a6eecb8b94" />

Yeah, that looks a lot better. You live and you learn. 
