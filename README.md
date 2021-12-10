from http.server import BaseHTTPRequestHandler, HTTPServer
from io import BytesIO
import sys
import string
import re
import urllib.parse
import time
import json 
import ssl



hostName = "localhost"
serverPort = 8080

class MyServertest(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers["Content-Length"])
        try:
            self.data_string = self.rfile.read(int(self.headers['Content-Length']))
            data= json.loads(self.data_string)
            result=0
            if(data['operation']=='add'):
                result=data['arguments'][0]+data['arguments'][1]
                print(result)
            elif(data['operation']=='sub'):
                result=data['arguments'][0]-data['arguments'][1]
                print(result)
            elif(data['operation']=='mul'):
                result=data['arguments'][0]*data['arguments'][1]
                print(result)
            elif(data['operation']=='divide'):
                if(int(data['arguments'][1])!=0):
                    result=format(float(data['arguments'][0])/float(data['arguments'][1]),".2f")
                else:
                    result="can't divide by zero"
        except:
            result="There is an error "
            print(result)
        Out= str(json.dumps({'result': str(result)}))
        self.send_response(500) #create header
        self.send_header("Content-Length",str(len((Out))))
        self.end_headers()
        self.wfile.write(str(Out).encode()) #send response           

    def do_GET(self):
        self.send_response(500)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title> http server </title></head>", "utf-8"))
        txt = str(self.path)
        txt = txt.upper()
        txt = txt.split("/")
        a = txt[2]
        b = txt[3]
        c = 0
        newresult = ""
        try:
            if txt[1] == "ADD":
                c = int(a) + int(b)
            elif txt[1] == "SUB":
                c = int(a) - int(b)
            elif txt[1] == "MUL":
                c = int(a) * int(b)
            elif txt[1] == "DIV":
                c = int(a) / int(b)
        except ZeroDivisionError:
            newresult = "You can't divide by Zero"
        else:
            newresult = 'message request :' + str(c)

        self.wfile.write(bytes(newresult, "utf-8"))
        # self.wfile.write(bytes("<p>Request: %s</p>" % self.path, "utf-8"))
        self.wfile.write(bytes("<body>" , "utf-8"))
        self.wfile.write(bytes("</body></html>","utf-8"))


if __name__ == "__main__":
    webServer = HTTPServer((hostName, serverPort), MyServertest)
    print("Server started http://%s:%s" % (hostName, serverPort))

    try:
        webServer.serve_forever()
    except RuntimeError:
        webServer.shutdown()
        sys.exit()
    except KeyboardInterrupt:
        pass

    webServer.server_close()
    print("Server stopped.")
