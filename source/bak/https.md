javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshake
finished
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1002)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1385)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1413)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1397)
	at sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:559)
	at sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:185)
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1546)
	at sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1474)
	at java.net.HttpURLConnection.getResponseCode(HttpURLConnection.java:480)
	at sun.net.www.protocol.https.HttpsURLConnectionImpl.getResponseCode(HttpsURLConnectionImpl.java:338)
	at webMagicTest.Test.downloadImage(Test.java:66)
	at webMagicTest.Test.main(Test.java:20)
Caused by: java.io.EOFException: SSL peer shut down incorrectly
	at sun.security.ssl.InputRecord.read(InputRecord.java:505)
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:983)
	... 11 more