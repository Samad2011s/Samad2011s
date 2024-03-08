import sys
import os
import cv2
import math
import speech_recognition as sr
import pyttsx3
import boto3
import pywhatkit
import requests
import smtplib
from email.message import EmailMessage
from twilio.rest import Client

# Initialize the text-to-speech engine
speak = pyttsx3.init()
voices = speak.getProperty('voices')
speak.setProperty('voice', voices[1].id)

# Define a function that takes voice commands and performs the corresponding actions
def process(text):
    # Your application goes here
    if 'take a picture' in text:
        image_cap()
    elif 'launch an EC2 instance' in text:
        ec2launch()
    elif 'listen' in text:
        listen(0)
    elif 'send a message' in text:
        sms()
    elif 'create a bucket' in text:
        create_s3_bucket()
    elif 'send a WhatsApp message' in text:
        whatsap()
    elif 'send an email' in text:
        email()
    elif 'calculate' in text:
        calculate()
    else:
        search_web(text)

# Define a function that takes voice commands and performs the corresponding actions
def listen(x):
    r = sr.Recognizer()
    if x == 0:
        talk('Hi. How can I help?')
    with sr.Microphone() as source:
        audio = r.listen(source)
    try:
        text = r.recognize_google(audio)
        y = process(text.lower())
        return(y)
    except:
        if x == 1:
            talk('Good Bye!')
        else:
            talk('I did not get that. Please say again.')
            listen(1)

# Define the image_cap function
def image_cap():
    cap = cv2.VideoCapture(0)
    status, pic = cap.read()
    cv2.imshow("hi", pic)
    cv2.waitKey()
    cv2.destroyAllWindows()

# Define the ec2launch function
def ec2launch():
    myec2 = boto3.client("ec2")
    response = myec2.run_instances(
        ImageId='ami-id',
        InstanceType="t2.micro",
        MaxCount=1,
        MinCount=1
    )

# Define the calculate function
def calculate():
    try:
        expression = input('Enter the mathematical expression: ')
        result = eval(expression)
        talk('The result is ' + str(result))
    except:
        talk('I cannot calculate that.')

# Define the search_web function
def search_web(query):
    try:
        url = 'https://www.google.com/search?q=' + query
        headers = {'User-Agent': 'Mozilla/5.0'}
        r = requests.get(url, headers=headers)
        talk('Here are the search results for ' + query)
        os.system('open ' + url)
    except:
        talk('I cannot find that.')

# Define the email function
def email():
    EMAIL_ADDRESS = os.environ.get('EMAIL_USER')
    EMAIL_PASSWORD = os.environ.get('EMAIL_PASS')
    msg = EmailMessage()
    msg['Subject'] = 'holla holla!!'
    msg['From'] = 'sender mail id'
    msg['To'] = 'receiver mail id'
