<p align="center"> 
  
![1_oSsQf_VmxRU-ApnVhz9-tQ](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/5b1002fe-9ef1-480b-aca2-87ec620e8638)

</p>

# Building-Intuition-For-DNS


In this Repo, I aim to show you and simplify the concept of DNS (Domain Name System) for an easier grasp of understanding it. We will cover topics such as DNS A-Records, the creation and deletion of records, CNAME Records, and Root Hints.


<h2>Environments and Technologies</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Microsoft Remote Desktop (Mac) (RDP)
- Command Line
- Microsoft Azure Active Directory (On-premises)
- Domain Name System (DNS)

<h2>Operating Systems</h2>

- Windows 10 (21H2)

<h2>List of Prerequisites</h2>

If you have completed https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs without closing out your session, you are good to go.
- If starting from scratch:
  - Client VM joined to your Domain
  - MicroSoft Azure w/ two pre-configured VM's
  - Client-01 and DC-1 Virtual Machines
  - MircoSoft Active Directory installed in DC-1
  - DC-1 promoted to a Domain Controller
  

![Screen Shot 2023-12-23 at 4 15 12 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/c269c7e1-0b49-4090-b4d2-d69d95d68b08)


<h2>Overview (What We're Going To Do)</h2>

1. Inspect DNS A-Records on the server (hostname to IP address mappings)
2. Create some of our own A-Records on the server and observe them from the client
3. Delete records from server and inspect/clear the client DNS cache to gain understanding
4. Touch on "CNAME" records (mapping one name to another name)
5. Discuss Root Hints: If you're using our local DNS server to resolve names into IP addresses, why can we go to Google and all the rest of the sites on the internet? Does our local DNS saerver know about those?

<h2>A-Records</h2>

A-Records are mappings from Host Names to IP Addresses

- Using the same credentials from https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs, go ahead and use Microsoft Remote Desktop to log into DC-1 as your domain admin account (mydomain.com\jane_admin)
- Connect and log into Client-1 as an admin (jane_admin)
  - From Client-1 try to ping "mainframe" and notice that it fails to do so

![Screen Shot 2023-12-23 at 6 50 44 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/4d0d0091-4919-4d58-bcd2-88dcaff0d970)

- This currently represents what took place behind the scenes on why we are getting the error message in our Command Prompt
  - It couldn't find a record in our cache, host file, or local DNS server

![Screen Shot 2023-12-23 at 6 49 30 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/e40d2e73-6e1b-4caa-83fe-46ba36e4787f)

- Switch over to DC-1 so we can get started on creating an A-Record for mainframe ("mainframe" is just a random name, we could have chosen "zebra" if we really wanted to)
- Go into your Server Manager dashboard and click on "Tools" and then "DNS" from the drop down menu

![Screen Shot 2023-12-23 at 6 55 51 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/efc6c164-f355-4e52-b0d7-00ff02091e61)

- Click and expand "DC-1".
  - Forward Lookup Zone: Host Name to IP address mapping
  - Reverse Lookup Zone: IP Address to Host Name mapping
- Expand "Forward Lookup Zone" and click on "mydomain.com"
  - You should now see the list of A-Records that we have right now
- Remember that an A-Record just means host name to IP address mapping
  - So you can see Client-1 as a Host Name and it maps to 10.0.0.5 while dc-1 is another Host Name that maps to 10.0.0.4 
- These automatically got registered over the network when Client-1 went online. DNS is integrated within Active Directory

![Screen Shot 2023-12-23 at 7 02 13 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/47eb1b24-0c1c-485d-8d06-862d946d0f1c)

- Manually create another A-Record for the host name "mainframe"
  - Right click and click on "New Host (A or AAAA)..."

![Screen Shot 2023-12-23 at 7 10 58 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/72d4ee7a-92f5-4cfa-b6b5-6ed9afd30bb5)

- Fill out the name and go ahead and choose dc-1's IP address then click "Add Host"
    - Essentially what we are doing right now is arbitrary. We are just randomly creating an A-Record so we can observe what happens on Client-1 when we do this
      
![Screen Shot 2023-12-23 at 7 12 55 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/4838dde8-f267-4d21-957b-b6077033ccba)

- As you can see we have successfully created an A-Record for mainframe
  - Now we can go back to Client-1 and try to ping mainframe again

![Screen Shot 2023-12-23 at 7 19 16 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/5fa2d75d-3757-4b03-8e8c-858b7ed490a8)

- You can see here that the ping succeeds this time around
- Type in "nslookup mainframe" on your command line
  - Observe how the dc-1 server with the 10.0.0.4 IP address was able to return the record "mainframe.mydomain.com"

![Screen Shot 2023-12-23 at 7 34 02 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/5e684c6b-cff8-4c3d-8ff5-a65963eb4fa4)

<h2>Local DNS Cache</h2>

- To demonstrate the DNS cache on Client-1, type in "ipconfig /displaydns" into the Command Prompt
  - This should show the records it has already gone to. When it resolved it once already, the record is stored in the cache
  - So for the next time we ping "mainframe", instead of bothering the DNS server, it's just going to use the local cache to look through the names Client-1 has figured out already because that is much faster
 
![Screen Shot 2023-12-23 at 8 10 01 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/faf35847-6828-4932-bdcb-a692b263f8d4)

- Now go back to DC-1's remote connection and change mainframe's record address to 8.8.8.8
  - Double click on the "mainframe" A-Record that we made
  - Update the ID address to "8.8.8.8" then hit "Apply" and "OK"

![Screen Shot 2023-12-23 at 9 02 24 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/3fc0279e-0efc-426b-bf4c-2e3d944b966a)
  
- Go back to Client-1 and ping mainframe again. Observe that it pings the new address
- Also observe the local dns cache (ipconfig /displaydns)

![Screen Shot 2023-12-23 at 9 25 21 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/ef851aa3-f305-4d1b-9cd0-d328654282b5)

- By any chance, if this IP address did not manage to update, many problems could occur
  - Sometimes a network's IP address might change but your local computer still has the old IP address cached which can cause you not to reach certain resources  
- Flush the DNS cache using "ipconfig /flushdns" as a local admin
  - Search up "Command Prompt", right click it, choose "Run as administrator", then run the command
- After you flush it you will see that the cache is now empty and that the updated IP address should now be in effect if you ping it once again

![Screen Shot 2023-12-23 at 10 19 36 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/ec0362cd-d030-4994-a575-bfb393724f57)


















- Open DC-1
- Right Click `mainframe`
- Select `Properties`

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/7abd0b4e-4e22-4792-91cf-52e1c29ed6bc)

- Change `IP Address` to **8.8.8.8**
- Click `Apply`
- Click `OK`

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/d3c65d72-403b-4b38-8a62-59d5f1c3c811)

- Return to Client-01
- In `Command Prompt` **ping mainframe**
- Notice it continues to ping the old address

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/08fb07a4-13a4-4a43-be14-021f830af9e8)

- Check the local DNS Cache using **ipconfig /displaydns**

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/3e855107-317b-4599-9fac-76c0bab5d039)

- Flush the DNS Cache using **ipconfig /flushdns**
- Observe the Cache is now empty

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/71e90d84-321b-4d62-9e34-6635746fa2a9)

- Type **ping mainframe**
- Observe the new IP Address shown

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/7c810328-ecab-4fb5-89f0-3c631e5b0aa6)

<h3>CNAME Record Exercise</h3>

>_A DNS CNAME record provides an alias for another domain._


- Return to DC-1
- Right click and Select `New Alias (CNAME)...`

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/083b6c9b-6543-421c-ab5a-5d699c19556f)

- Set `Alias name` to **search**
- Set `Fully qualified domain name` to **www.google.com**
- Select `OK`

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/647d0220-f27d-4c1a-8139-e3c88d944bdb)

- Return to Client-01
- Type **ping search**
- Observe the results of CNAME Records

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/79fa1b1e-6ff2-4d5c-980d-07852d0ec310)

- Type **nslookup mainframe** to query mainframe

![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/36c10303-75a9-4670-a25b-2693a9d67bfa)

- Type **nslookup search** to query search
  
![image](https://github.com/CarlosAlvarado0718/DNS-Intuition/assets/140138198/9673db75-615e-4a2e-9230-8c4805c50857)

<h1>ALL DONE!!! CONGRATS!!!</h1>
<h3>Now you have a better understanding of DNS, A-Records, DNS caching and CNAME Records. </h3>
