************* ju wang note on gstreamer, with ros gscam usage *************

8/28/2017

sending:
alias videofileudpstream2='python ~/turtlebot/src/rqt_mypkg/scripts/start3.py "use the same sprop-parameter-sets at the receiver" && roscd gscam/examples; gst-launch-1.0 -v filesrc location = video.mp4 ! qtdemux ! h264parse ! rtph264pay ! udpsink host=localhost port=5600'

receiving:
alias videoudpsink2=' gst-launch-1.0 udpsrc port=5600 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, sprop-parameter-sets=(string)\"Z0LAHp4hgRhTTUBAQFAAAAMAEAAAAwPI8WLu\,aM4GyyA\\=\"" ! rtph264depay ! avdec_h264 ! autovideosink sync=false'
------------------------------------
videostreamh264=' gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,width=640,height=480,framerate=15/1 ! x264enc pass=qual quantizer=20 tune=zerolatency ! rtph264pay ! udpsink host=localhost port=5600'

videosinkh264='gst-launch-1.0 -e -v udpsrc port=5600 ! application/x-rtp, payload=96 ! rtpjitterbuffer ! rtph264depay ! avdec_h264 ! autovideosink sync=false'

videostreamh264tofile='gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,width=640,height=480,framerate=15/1 ! x264enc pass=qual quantizer=20 tune=zerolatency ! flvmux ! filesink location=test.flv’

videostreammjpeglocal='gst-launch-1.0 v4l2src device=/dev/video1 ! image/jpeg,width=640,height=480,framerate=15/1 ! jpegdec ! xvimagesink'
	This is for microsoft camera (output mjpeg and yuv), localview

videostreammjpeg='gst-launch-1.0 v4l2src device=/dev/video1 ! image/jpeg,width=640,height=480,framerate=15/1 !  rtpjpegpay  ! udpsink host=localhost port=5600'
	This is for microsoft camera (output mjpeg and yuv), stream out mjpeg

videosinkjpeg='gst-launch-1.0 -e -v udpsrc port=5600 ! application/x-rtp, payload=96 ! rtpjitterbuffer ! rtpjpegdepay ! jpegdec ! autovideosink sync=false'
obtain mjpeg from udp and view

Using test video src:
gst-launch-1.0 videotestsrc pattern=ball ! x264enc ! rtph264pay ! udpsink host=127.0.0.1 port=5600
 roslaunch videoudpmjpeg.launch

--------------------------
Working combinations:
Videostreammjpeg -> videosinkjpeg: microsoft camera, mjpeg, udp 5600.
Microsoft camera ->Videostreammjpeg -> videoudpmjpeg.launch -> rqt-image view
videostreamh264 -> videosinkh264
videostreamh264 -> qgc
videofileudpstream2->videoudpsink2
solo -> qgc
solo -> videosinkh264+qgc
solo -> videosinkh264+ “telnet 10.1.1.1 5502”,  video not show if gopro not plugged
solo -> ros panel click "telnet solo" + "check mjpeg/h264" + videofrom5600 + image_view,  video not show if gopro not plugged
TBD: file -> qgc, file has a sprop parameters, which is not present in qgc
Qgc send something to solo to keep streaming...
Gscam videofile.launch -> rqt_image_view

Not working yet:
Videostreamh264 → gscam videoudp ->rqt_image_view
Gscam using gstreamer 0.1, need to rebuild to link to 1.0 for testing.

Buggy trouble shooting:
Killall gscam, kill gst-launch-1.0, 

