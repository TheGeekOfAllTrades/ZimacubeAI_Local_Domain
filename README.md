# Setup Zimacube AI with a Local Domain

**What is needed for this project:**
- Zimacube or Zimacube pro
- Nvidia RTX A2000 12gb, RTX 2000 ADA, RTX 2000e, or similar small form factor GPU
- PiHole DNS Service application
- NGINX reverse proxy
- Compute host for DNS & Reverse proxy

# **Setup the Zimacube for AI Apps**
- Setup and log into Zimacube, get it up and running as your system. 
- Power down, and install RTX A2000 GPU, then power back on. 
- Visit app store and download WebUI app
  
![image](https://github.com/user-attachments/assets/b9d1429a-8abd-42e4-b08a-7abf1831929a)

- Open WebUI app and create a new primary user
  
![image](https://github.com/user-attachments/assets/8f5464c1-e121-4130-8754-488743d4578b)

-- email really shouldn't matter, I haven't received an email from them and this is just used for login purposes.
- First login will likely show release notes for the updated version you are using

![image](https://github.com/user-attachments/assets/5fc9dceb-8fa9-49e1-94f9-813ac6b0fdee)

- You will not have LLM models already available, you have to download them.

![image](https://github.com/user-attachments/assets/99c6a7f2-d124-41ac-98b6-a1af9d9ec9d5)

- Visitn www.ollama.com and click the Models

![image](https://github.com/user-attachments/assets/37018a8d-2432-4842-9cf1-5c853be1ab06)

- From here, you can search or pick models you want to use. For now, I would say just pick a small one like a 1b or 3b model. You can always grab more later, but let's finish the setup.

![image](https://github.com/user-attachments/assets/391d8b49-88ed-48ce-abf7-3966372cdf4d)

- Click the model and chose the parameter version if applicable, then copy the model name with parameter model size. "qwen:1.8b" is our example here.

![image](https://github.com/user-attachments/assets/2cc6b190-89e0-42e7-8590-1106893192f9)

- Paste the model name into the model search bar, then click pull "model? from ollama.com and wait for download to finish.

![image](https://github.com/user-attachments/assets/8856318c-13d7-463b-bf6b-6a619efc5579)
-- this could take a while, depending on the size of the model and your internet bandwidth

- Once downloaded, it will verify the model and BOOM! You now have a working AI model living in your LAN.

![image](https://github.com/user-attachments/assets/47d5bed9-de88-4331-aa05-e34597426b7c)

# **Migrate App Data Off Primary SSD**
- By default, you apps and app data will reside on the included 256gb NVMe drive, which will fill up quickly with models.
- Get a larger capacity, 1TB or bigger, to install in the NVMe Zimacube tray.
- After installed and provisioned, click the storage settings icon from the main ZimaOS dashboard

![image](https://github.com/user-attachments/assets/a161b634-24fe-4218-be0a-c0bc1a2392db)

- Click Apps on the next menu, the the migration icon next to App Data

![image](https://github.com/user-attachments/assets/a0e9d448-830d-42ce-95ba-4d863aedcf23)

- Then select the NVMe drive you want to move the data onto, and select next.

![image](https://github.com/user-attachments/assets/acb5769b-7b2a-4ba7-87e4-b1257f51cc83)

- From there, select the checkbox signaling your awareness of this change and time involved, then hit start migrating

![image](https://github.com/user-attachments/assets/6eb645cd-59a7-415d-8ef2-c22a5ab264ff)

- Now, you have WAY more room to download new models and play with your new WebUI private AI tool


# **Publish the WebUI to a .local Domain Name**
## Configure PiHole DNS
- You will need some kind of physical infrastructure here, to host PiHole. I prefer Proxmox as a hypervisor, so I can use the same host for multiple VMs or containers.
{in this example, I use Proxmox running an Ubuntu VM to host PiHole, visit www.pi-hole.net for docker or OS type install instructions}
- When you install PiHole on your device, thru the installation screens it will ask if you want to install the Admin console, **be sure to click yes to this!**
- Set static IP address in your preferred method. At the OS level, router level or app level.
- Once completed, in a browser window input the IP address you set for the PiHole with /admin after, this will take you to the admin dashboard.

![image](https://github.com/user-attachments/assets/109c294e-7f3d-47da-ab00-aa4826eb8a61)
-- You don't have to add the /index.php in the url, it will auto populate that way

## Configure NGINX Reverse Proxy
- Using your preferred device, create an OS to hold the NGINX app
{I am using an LXC container within Proxmox}
- Visit www.nginx.org for instructions to install on your preferred OS.
- Be sure to set static IP address for this nginx server, like you did with PiHole in your preferred method. 
- Visit your ZimaOS WebUI app, and write down the IP address and Port number. 
- Once installed, SSH into the NGINX or visit terminal and create configuration
  - nano /etc/nginx/sites-available/(app name)
- Input this config, changing the app name and ip address:port for your setup
  - server {
    listen 80;
    server_name (app name).local;

    location / {
        proxy_pass http://(IP address:port);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
- Exit nano by ctrl+x, Y to save and Enter to keep the file name.
- Then modify this script for your setup and run it
  - ln -s /etc/nginx/sites-available/(app name) /etc/nginx/sites-enabled/
- Test the configuration with
  - nginx -t

![image](https://github.com/user-attachments/assets/f2927aed-5fda-4182-94fb-540756871817)

- When return shows Test is Successful, restart the service
  - systemctl restart nginx
- Verify reverse proxy by running this
  - curl -I http://(app name).local
- This should return HTTP 200 OK, plus some more stats about your NGINX server

## Finish the Setup
- Now let's tie it all together
- Go back to your PiHole admin dashboard, and click Local DNS on the side menu, then DNS Records in the dropdown menu.

![image](https://github.com/user-attachments/assets/61983816-b95c-4813-96bc-8c5a8cc1763b)

- From here, input your chosen (app name).local domain and the ip address of the NGINX server, then click add
  - You can see the example of the domain names I created for my setup in this screenshot, you can name them whatever you want as long as they match the config you created in NGINX

![image](https://github.com/user-attachments/assets/0eaf8b39-55a9-4387-b84e-13b89ae3154f)

- For testing, before you push this into full network production, use a local machine or test machine and change the DNS from that workstation to the PiHole IP address. I'm using a Windows 11 VM as an example here.

![image](https://github.com/user-attachments/assets/8bebac4a-baca-41f0-ad26-e8abdf125265)

- Next, go back to your ZimaOS dashboard and get back into WebUI, then log out from that instance of WebUI
- Back at your VM or test workstation, Open a browser window and put your new .local domain into the url bar.
- From here, you can login with your existing creds to continue where you left off, with full history and preference settings published to that browser!

![image](https://github.com/user-attachments/assets/85e2a4d7-976f-4c0c-b269-50d0be46d6b4)

- You can also create multiple users in the WebUI dashboard as admin for remote access to the WebUI app as well, whether its other users in the LAN or for yourself for various reasons.


# **IF THIS WAS HELPFUL, PLEASE SUBSCRIBE TO MY YOUTUBE CHANNEL AT WWW.YOUTUBE.COM/@GEEKOFALLTRADES**
