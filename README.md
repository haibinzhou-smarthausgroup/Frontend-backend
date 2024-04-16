# Frontend-backend
This repository explains how frontend and backend work together
## How Frontend use voice as input/output
Client use Javascript with Audio Worklet for voice input/output
Speech is recorded into chunks of audio files with 5 seconds duration. Speech record automatically stop if user is silent for 3 seconds.
User can use send button to stop recording and send audio file to API gateway over Websocket connection.
The audio file is in binary format that cannot work directly with Lambda because Lambda only can process JSON data.
We have to convert audio file to Base64 first in Javascript. Lambda will convert it back to binary.
AWS API Gateway only support message frame size less than 32 KB. So the audio file must be splited into files that is smaller than 32 KB in size.

The audio chunks will be sent to API gateway with we.send(chunk). Each chunk will trigger separate Lambda instance. It works in ansync mode.
Lambda will use Whisper to convert audio to text. The issue is it will take different time for Whisper to send back the text.
The second chund conversion result may be sent back to Lambda befor first chunk. It can be out of order.
We must keep the correct order for chunk results. It is about keep correct order between multiple Lambda functions.
We have to use DynamoDB for this purpose.
DynamoSB will have table named speech_chunk
Partation key is connectionID
sort key is chunk dequence #

Client side Javescript will keep send chunks until the recording is stopped, just like user enter text and clik send button. So AI can generate correct answer based on the full questions.
Client will send chunk_finish message to different Lambda function. This function will search DynamoDB to find chunks in correct order. Combine them together in Base64 format, then convert Base64 file to binary file.
Then this Lambda function will send the file to Whisper, get the text result back, send the text back to client to show.
Then Lambda send this text as query to AI and get the answer.
Also this Lambda will use Whisper to convert the answer to voice. Split it in chunks less than 32 KB and convert it to Base64 format. Send the chunks to client.
Client combine the chunks into one large file and play it back with Audio Worklet.
