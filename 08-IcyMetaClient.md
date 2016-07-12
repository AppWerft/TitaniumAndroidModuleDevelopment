#Ti.IcyMetaClient

The sound file format embeds [ID3Tags](https://en.wikipedia.org/wiki/ID3) to deliver meta details about the trace. These data are in the tail of file. In field of live streaming is no end. Therefore the meta data will send on start of streaming and during streaming. 

![](http://www.smackfu.com/stuff/programming/shoutcast.gif)

If we start the streaming we send a special http header and then the server sends these meta data between the mp3/aac data in fixe distances. More you can find [here](http://www.smackfu.com/stuff/programming/shoutcast.html)


