import cv2
import numpy as np
import os
import time
import sys
from datetime import datetime

import queue
import threading
import random
import requests

from weakref import WeakValueDictionary


IMAGES_FOLDER = "images/"
IMAGES_PREFIX = "_IMAGE_"
    
SENTINEL = "END"
NUM_THREADS = 1
q = queue.Queue()
workers = []

class FrameObj(object):
    #_instances = WeakValueDictionary()
    #@property
    #def Count(self):
    #    return len(self._instances)
    
    def __init__(self, frame_time, curr_time_float, cap_frame, cap_title=""):
        self.frame_time = frame_time
        self.frame_cap =  cap_frame
        self.frame_time_str = str(frame_time.strftime("D_%Y-%m-%d-_T_%H_%M_%S_%f"))
        self.cap_title = cap_title
        self.curr_time_float = curr_time_float
     #   self._instances[id(self)] = self

    def __del__(self):
        self.cleanup()
                
    def cleanup(self):
        self.frame_cap = None
        self.frame_time = ""
        self.frame_time_str = ""
        self.cap_title = ""
        
    def __str__(self):
        return '<' + self.frame_time_str + '>'

    def __repr__(self):
        return '<' + self.frame_time_str +'>'

    def get_frame_time(self):
        return self.frame_time

    def get_frame_cap(self):
        return self.frame_cap

    def get_cap_title(self):
        return self.cap_title

    def get_frame_time_float(self):
        return self.curr_time_float
    
def save_image_cv_frame(queue):
    prvs_time = time.time_ns()
    while True:
        frame = queue.get()
        if(frame.get_cap_title() != SENTINEL):            
            filename = IMAGES_FOLDER + IMAGES_PREFIX + str(frame.get_frame_time().strftime("D_%Y-%m-%d_T_%H_%M_%S_%f"))  + ".jpg"
            if(frame.get_frame_cap() is not None):
                cv2.imwrite(filename, frame.get_frame_cap())
            queue.task_done()
  
            print("Save image: ", frame)
            #print("Time image: ", ((frame.get_frame_time_float() - prvs_time)*1000.0))
            print("Time image: ", ((frame.get_frame_time_float() - prvs_time)/ (10 ** 6)))
            prvs_time = frame.get_frame_time_float()
            frame.cleanup()
            frame = None
        else:
            print("Retrieved element END sentinel...")
            queue.task_done()
            break
        
    print("save_image_cv_frame Thread exited...")


for i in range(NUM_THREADS):
    worker = threading.Thread(target=save_image_cv_frame,args=(q,))
    worker.start()
    workers.append(worker)

def null(*args):
    pass

cv2.namedWindow("MainWindow")
#cv2.setMouseCallback('MainWindow',process_click)
#cv2.createButton("Quit",null,None,cv2.QT_PUSH_BUTTON,1)
#cv2.createTrackbar("Exit App", "MainWindow", 0, 1, null)


#os.environ["OPENCV_FFMPEG_CAPTURE_OPTIONS"] = "rtsp_transport;udp"

rtsp_url = "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov"
fps = 50

(major_ver, minor_ver, subminor_ver) = (cv2.__version__).split('.')

print("Trying to connect: ", rtsp_url)

try:
    vcap = cv2.VideoCapture(rtsp_url)

    if(vcap is None):
        print("Failed to connect to the requested stream.");
        exit

    print("Connected to: ", rtsp_url)
    print("Saving to: ", IMAGES_FOLDER)

    if int(major_ver)  < 3 :
        fps = vcap.get(cv2.cv.CV_CAP_PROP_FPS)
        print ("Frames per second using video.get(cv2.cv.CV_CAP_PROP_FPS): {0}".format(fps))
    else :
        fps = vcap.get(cv2.CAP_PROP_FPS)
        if( fps <= 0 ):
            fps = 1
        
    print ("Frames per second using video.get(cv2.CAP_PROP_FPS) : {0}".format(fps))

    frame_width = int(vcap.get(3))
    frame_height = int(vcap.get(4))    

    print ("Frame Dimension: {0} X {1}".format(frame_width, frame_height))
    # Image Frame Per Second
    imageProcessingInterval = (1000.0 / fps)

    # when we last procesed an image
    lastProcessed = time.time_ns()
    print("Init Time: ", lastProcessed , ", Interval= ", imageProcessingInterval, ", FPS= ", fps)

    while(1):
        ret, frame = vcap.read()
        
        curr_time = time.time_ns()
        #elapsed_ms = ((curr_time-lastProcessed)*1000.0)
        elapsed_ms = ((curr_time-lastProcessed) / (10 ** 6))
        #print("Curr: ", curr_time, "Last: ", lastProcessed, ",Elapsed: ", elapsed_ms)
        #if ((lastProcessed + imageProcessingInterval)) >= time.time():
        if(elapsed_ms >= imageProcessingInterval):
            lastProcessed = time.time_ns()

            if ret == False:
                print("Frame is empty")
                break;
            else:
                q.put(FrameObj(datetime.now(), curr_time, frame, ""))
                #filename = imagesFolder + "/image_" + str(datetime.now().strftime("D_%d-%m-%Y_T_%H_%M_%S_%f"))  + ".jpg"
                #print(filename)
                #cv2.imwrite(filename, frame)
                cv2.putText(frame,"ESC - QUIT",(width - 200,20), FONT, 1 ,(255,255,255))
                cv2.imshow('MainWindow', frame)
                cv2.waitKey(1)
except:
    e = sys.exc_info()[0]
    print("Unexpected error with RTSP: %s" % e)


q.put(FrameObj(datetime.now(), curr_time, None, SENTINEL))
vcap.release()
print ("Done!")


for w in workers:
    w.join()

cv2.destroyAllWindows()
