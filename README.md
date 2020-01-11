# Eger
Egércsapda kód

#!/usr/bin/python

import RPi.GPIO as GPIO
import subprocess
import time
import picamera
import smtplib,ssl
import board
import neopixel

from picamera import PiCamera
from time import sleep
from email.mime.multipart import MIMEMultipart  
from email.mime.base import MIMEBase  
from email.mime.text import MIMEText  
from email.utils import formatdate  
from email import encoders
from subprocess import call


GPIO.setmode(GPIO.BCM)

infra=23
servo=24
pixels = neopixel.NeoPixel(board.D18, 8)

GPIO.setwarnings(False)
GPIO.setup(23,GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.setup(24,GPIO.OUT)
camera=PiCamera()
pwm = GPIO.PWM(24, 50) # Szervo def.
pwm.start(7.5)   # Szervo zárás

while True:
   if GPIO.input(23)==0:
       pwm.ChangeDutyCycle(10.5)
       time.sleep(1)
       pwm.stop()
       GPIO.cleanup()
       time.sleep(2)
   
       for x in range(0, 8):   #Lámpa LED-ek 
           pixels[x] = (255, 255, 255)  # Szín fehér
       
       camera.start_preview()  
       sleep(2)  
       camera.capture('/home/pi/image.jpg')     
       sleep(3)  
       camera.stop_preview()
       
       for x in range(0, 8):
           pixels[x] = (0, 0, 0)
           
       def send_an_email():  
           toaddr = 'ikkert14@gmail.com'
           me = 'ikkert14@gmail.com'         
           subject = "Megfogtam az egeret"              # Tárgy
  
           msg = MIMEMultipart()  
           msg['Subject'] = subject  
           msg['From'] = me  
           msg['To'] = toaddr  
           msg.preamble = "Eger fenykep mellekelve "  
           part = MIMEBase('application', "octet-stream")  
           part.set_payload(open("image.jpg", "rb").read())  
           encoders.encode_base64(part)  
           part.add_header('Content-Disposition', 'attachment; filename="image.jpg"')   # Fájl és formátum név
           msg.attach(part)  
  
           try:  
              s = smtplib.SMTP('smtp.gmail.com', 587)  # Protocol
              s.ehlo()  
              s.starttls()  
              s.ehlo()  
              s.login(user = 'ikkert14@gmail.com', password = 'idenemlepszbe')  # Felhasználó & jelszó
              s.sendmail(me, toaddr, msg.as_string())  
              s.quit()     
           except SMTPException as error:  
                 print ("Error")                  
       send_an_email()
       sleep(3)
       break

#os.system('sudo poweroff')
