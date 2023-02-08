# keyboard
import cv2
import numpy as np 
import time
from keys import *
from handTracker import *
from pynput.keyboard import Controller

def getMousPos(event , x, y, flags, param):
    global clickedX, clickedY
    global mouseX, mouseY
    if event == cv2.EVENT_LBUTTONUP:
        #print(x,y)
        clickedX, clickedY = x, y
    if event == cv2.EVENT_MOUSEMOVE:
    #     print(x,y)
        mouseX, mouseY = x, y

def calculateIntDidtance(pt1, pt2):
    return int(((pt1[0]-pt2[0])**2 + (pt1[1]-pt2[1])**2)**0.5)

# Creating keys
w,h = 80, 60
startX, startY = 40, 200
keys=[]
letters =list("QWERTYUIOPASDFGHJKLZXCVBNM")
for i,l in enumerate(letters):
    if i<10:
        keys.append(Key(startX + i*w + i*5, startY, w, h, l))
    elif i<19:
        keys.append(Key(startX + (i-10)*w + i*5, startY + h + 5,w,h,l))  
    else:
        keys.append(Key(startX + (i-19)*w + i*5, startY + 2*h + 10, w, h, l)) 

keys.append(Key(startX+25, startY+3*h+15, 5*w, h, "Space"))
keys.append(Key(startX+8*w + 50, startY+2*h+10, w, h, "clr"))
keys.append(Key(startX+5*w+30, startY+3*h+15, 5*w, h, "<--"))

showKey = Key(300,5,80,50, 'Show')
exitKey = Key(300,65,80,50, 'Exit')
textBox = Key(startX, startY-h-5, 10*w+9*5, h,'')

cap = cv2.VideoCapture(0)
ptime = 0

# initiating the hand tracker
tracker = HandTracker(detectionCon=0.8)

# getting frame's height and width
frameHeight, frameWidth, _ = cap.read()[1].shape
showKey.x = int(frameWidth*1.5) - 85
exitKey.x = int(frameWidth*1.5) - 85
#print(showKey.x)

clickedX, clickedY = 0, 0
mousX, mousY = 0, 0

show = False
cv2.namedWindow('video')
counter = 0
previousClick = 0

keyboard = Controller()
while True:
    if counter >0:
        counter -=1
        
    signTipX = 0
    signTipY = 0

    thumbTipX = 0
    thumbTipY = 0

    ret, frame = cap.read()
    if not ret:
        break
    frame = cv2.resize(frame,(int(frameWidth*1.5), int(frameHeight*1.5)))
    frame = cv2.flip(frame, 1)
    #find hands
    frame = tracker.findHands(frame)
    lmList = tracker.getPostion(frame, draw=False)
    if lmList:
        signTipX, signTipY = lmList[8][1], lmList[8][2]
        thumbTipX, thumbTipY = lmList[4][1], lmList[4][2]
        if calculateIntDidtance((signTipX, signTipY), (thumbTipX, thumbTipY)) <50:
            centerX = int((signTipX+thumbTipX)/2)
            centerY = int((signTipY + thumbTipY)/2)
            cv2.line(frame, (signTipX, signTipY), (thumbTipX, thumbTipY), (0,255,0),2)
            cv2.circle(frame, (centerX, centerY), 5, (0,255,0), cv2.FILLED)

    ctime = time.time()
    fps = int(1/(ctime-ptime))

    cv2.putText(frame,str(fps) + " FPS", (10,20), cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0,0,0),2)
    showKey.drawKey(frame,(255,255,255), (0,0,0),0.1, fontScale=0.5)
    exitKey.drawKey(frame,(255,255,255), (0,0,0),0.1, fontScale=0.5)
    cv2.setMouseCallback('video', getMousPos)

    if showKey.isOver(clickedX, clickedY):
        show = not show
        showKey.text = "Hide" if show else "Show"
        clickedX, clickedY = 0, 0

    if exitKey.isOver(clickedX, clickedY):
        #break
        exit()

    #checking if sign finger is over a key and if click happens
    alpha = 0.5
    if show:
        textBox.drawKey(frame, (255,255,255), (0,0,0), 0.3)
        for k in keys:
            if k.isOver(mouseX, mouseY) or k.isOver(signTipX, signTipY):
                alpha = 0.1
                # writing using mouse right click
                if k.isOver(clickedX, clickedY):                              
                    if k.text == '<--':
                        textBox.text = textBox.text[:-1]
                    elif k.text == 'clr':
                        textBox.text = ''
                    elif len(textBox.text) < 30:
                        if k.text == 'Space':
                            textBox.text += " "
                        else:
                            textBox.text += k.text
                            
                # writing using fingers
                if (k.isOver(thumbTipX, thumbTipY)):
                    clickTime = time.time()
                    if clickTime - previousClick > 0.4:                               
                        if k.text == '<--':
                            textBox.text = textBox.text[:-1]
                        elif k.text == 'clr':
                            textBox.text = ''
                        elif len(textBox.text) < 30:
                            if k.text == 'Space':
                                textBox.text += " "
                            else:
                                textBox.text += k.text
                                #simulating the press of actuall keyboard
                                keyboard.press(k.text)
                        previousClick = clickTime
            k.drawKey(frame,(255,255,255), (0,0,0), alpha=alpha)
            alpha = 0.5
        clickedX, clickedY = 0, 0        
    ptime = ctime
    cv2.imshow('video', frame)

    ## stop the video when 'q' is pressed
    pressedKey = cv2.waitKey(1)
    if pressedKey == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()

Program 2:
import cv2 as cv 
import numpy as np 
import imutils
import json
import pyautogui
import time
cam = cv.VideoCapture(0)
arr=[]
nums=["1","2","3","4","5","6","7","8","9","0"]
row1=["Q","W","E","R","T","Y","U","I","O","P"]
row2=["A","S","D","F","G","H","J","K","L"]
row3=["Z","X","C","V","B","N","M"]
row4=["space","enter","backspace","shift"]
row5=["left","up","down","right"]
row6=["volumeup","volumedown","volumemute"]
x=10
y=20
for i in range(0,10):
    data={}
    data["x"]=x
    data["y"]=y
    data["w"]=100
    data["h"]=80
    data["value"]=nums[i]
    arr.append(data)
    x=x+100
y=100
x=10
for i in range(0,10):
    data={}
    data["x"]=x
    data["y"]=y
    data["w"]=100
    data["h"]=80
    data["value"]=row1[i]
    arr.append(data)
    x=x+100
x=110
y=180
for i in range(0,9):
    data={}
    data["x"]=x
    data["y"]=y
    data["w"]=100
    data["h"]=80
    data["value"]=row2[i]
    arr.append(data)
    x=x+100
x=210
y=260
for i in range(0,7):
    data={}
    data["x"]=x
    data["y"]=y
    data["w"]=100
    data["h"]=80
    data["value"]=row3[i]
    arr.append(data)
    x=x+100
x=110
y=340
data={}
data["x"]=x
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row4[0]
arr.append(data)
data={}
data["x"]=310
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row4[1]
arr.append(data)
data={}
data["x"]=510
data["y"]=340
data["w"]=250
data["h"]=80
data["value"]=row4[2]
arr.append(data)
data={}
data["x"]=760
data["y"]=340
data["w"]=200
data["h"]=80
data["value"]=row4[3]
arr.append(data)
x=110
y=420
data={}
data["x"]=x
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row5[0]
arr.append(data)
data={}
data["x"]=310
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row5[1]
arr.append(data)
data={}
data["x"]=510
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row5[2]
arr.append(data)
data={}
data["x"]=710
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row5[3]
arr.append(data)
x=10
y=500
data={}
data["x"]=x
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row6[0]
arr.append(data)
data={}
data["x"]=210
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row6[1]
arr.append(data)
data={}
data["x"]=410
data["y"]=y
data["w"]=200
data["h"]=80
data["value"]=row6[2]
arr.append(data)

json_string=json.dumps(arr)
json_data=json.loads(json_string)
#print(json_data)
while(1):
    ret,img = cam.read()
    img = cv.GaussianBlur(img,(5,5),0)
    img=imutils.resize(img,width=1030,height=700)
    height,width=img.shape[:2]
    x=10
    y=20
    for i in range(0,10):
        cv.rectangle(img,(x,y),(x+100,y+80),(0,255,255),2)
        cv.putText(img,nums[i],(x+50,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
        x=x+100
    y=100
    x=10
    for i in range(0,10):
        cv.rectangle(img,(x,y),(x+100,y+80),(0,255,255),2)
        cv.putText(img,row1[i],(x+50,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
        x=x+100
    x=110
    y=180
    for i in range(0,9):
        cv.rectangle(img,(x,y),(x+100,y+80),(0,255,255),2)
        cv.putText(img,row2[i],(x+50,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
        x=x+100
   
    x=210
    y=260
    for i in range(0,7):
        cv.rectangle(img,(x,y),(x+100,y+80),(0,255,255),2)
        cv.putText(img,row3[i],(x+50,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
        x=x+100
    x=110
    y=340
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row4[0],(x+70,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=310
    y=340
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row4[1],(x+70,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=510
    y=340
    cv.rectangle(img,(x,y),(x+250,y+80),(0,255,255),2)
    cv.putText(img,row4[2],(x+70,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=760
    y=340
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row4[3],(x+70,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=110
    y=420
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row5[0],(x+100,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=310
    y=420
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row5[1],(x+100,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=510
    y=420
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row5[2],(x+100,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=710
    y=420
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,row5[3],(x+100,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=10
    y=500
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,"V+",(x+60,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=210
    y=500
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,"V-",(x+60,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)
    x=410
    y=500
    cv.rectangle(img,(x,y),(x+200,y+80),(0,255,255),2)
    cv.putText(img,"Vmute",(x+60,y+40),cv.FONT_HERSHEY_SIMPLEX,1,(0,0,255),2,cv.LINE_AA,False)

    hsv_img = cv.cvtColor(img,cv.COLOR_BGR2HSV)
    mask = cv.inRange(hsv_img,np.array([65,60,60]),np.array([80,255,255]))
    mask_open = cv.morphologyEx(mask,cv.MORPH_OPEN,np.ones((5,5)))
    mask_close = cv.morphologyEx(mask_open,cv.MORPH_CLOSE,np.ones((20,20)))
    mask_final = mask_close
    conts,_ = cv.findContours(mask_final.copy(),cv.RETR_EXTERNAL,cv.CHAIN_APPROX_SIMPLE)
    #cv.drawContours(img,conts,-1,(255,0,0),3)
    if(len(conts)==1):
        x,y,w,h = cv.boundingRect(conts[0])
        cx = round(x+w/2)
        cy = round(y+h/2)
        cv.circle(img,(cx,cy),20,(0,0,255),2)
        word=""
        for i in range(len(json_data)):
            if cx>=int(json_data[i]["x"]) and cx<=int(json_data[i]["x"])+int(json_data[i]["w"]) and cy>=int(json_data[i]["y"]) and cy<=int(json_data[i]["y"])+int(json_data[i]["h"]):
                word=json_data[i]["value"]
        #print(word)
        pyautogui.press(word)
        #pyautogui.PAUSE=2.5
        #time.sleep(1)

    cv.imshow("cam",img)
    cv.waitKey(10)
