# BLUE HENS UDCTF
## Super Admin

https://bluehens-web-cookies.chals.io/

```category: WEB```
```challange points: 70```
```Description:Comfort food.```

![Alt text](image.png)

At Login page we see three options, ```user``` where we can register and ```profile``` where we can see content about the user. In ```chrome```>```DevTools```>```Application``` we can see a ```JWT Token``` with value ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoidXNlciIsImlhdCI6MTY5ODgyNDc0Mn0.OBuSPu7cUi9HFbL8OmiZpidVmBt5yndopULwmMStq7U``` is generated as ```cookies``` when we register on ```User```. Let's see the content of ```Jwt-Token``` on https://jwt.io/ 

![Alt text](image-1.png)

Here ```algroith:HS256``` is used and ```role:user```with a ```secret key```. So here our work is to change ```role``` to ```admin``` with correct ```secret key```. A basic attack of ```weak secret key``` is popular, in which we can bruteforce ```secret``` let's try that. 
you can use bruteforcing tool ```hashcat``` to do this. Open terminal and run commmand as 
```hashcat -a 0 -m 16500 eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoidXNlciIsImlhdCI6MTY5ODU3OTQzOH0.Xj3ZIxegIdGOm6FHmvrFO6g2hYqQnDh9cIdntOhvJ1s /home/h4l0gen/wordlists/rockyou.txt --show``` , i used ```rockyou.txt``` , it contains lot of passwords. So in seconds it will give you ```secret``` as ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoidXNlciIsImlhdCI6MTY5ODU3OTQzOH0.Xj3ZIxegIdGOm6FHmvrFO6g2hYqQnDh9cIdntOhvJ1s:password1```.

Now let us change user value to ```admin ``` and ```secret``` as ```password1```.

![Alt text](image-2.png)

copy this JWT Token and in ```chrome```>```DevTools```>```Application```> ```creds``` change its value to ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4iLCJpYXQiOjE2OTg4MjQ3NDJ9.lfHqMHpwxddAOLtAP_tnZmKrRI120rBEihE3rvI9yEA``` and reload page .

![Alt text](image-3.png)


Now go to ```profile``` section again 





![Alt text](image-4.png)  

and we found the flag as 
**Here is your flag: UDCTF{k33p_17_51mp13_57up1d_15_4_l1e}**

**conclusion: its a basic attack of weak secret key where we bruteforce it and then change session token value**
