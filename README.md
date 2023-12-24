<p align="center"> 
  
![1_oSsQf_VmxRU-ApnVhz9-tQ](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/5b1002fe-9ef1-480b-aca2-87ec620e8638)

</p>

# Building Intuition For DNS


In this repository, my goal is to demonstrate and simplify the concept of DNS (Domain Name System) for a more accessible understanding. We'll explore areas like A-Records, the creation and updating of records, CNAME, and Root Hints.

<h2>Overview (What We're Going To Do)</h2>

1. Inspect DNS A-Records on the server (hostname to IP address mappings)
2. Create some of our own A-Records on the server and observe them from the client
3. Update records from server and inspect/clear the client DNS cache to gain understanding
4. Touch on "CNAME" records (mapping one name to another name)
5. Discuss Root Hints: If you're using our local DNS server to resolve names into IP addresses, why can we go to Google and all the rest of the sites on the internet? Does our local DNS server know about those?

<h2>Environments and Technologies</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Microsoft Remote Desktop (Mac) (RDP)
- Microsoft Azure Active Directory (On-premises)
- Domain Name System (DNS)

<h2>Operating Systems</h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>List of Prerequisites</h2>

If you have completed https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs without closing out your session, you are good to go.
- If starting from scratch:
  - Client VM joined to your Domain
  - MicroSoft Azure w/ two pre-configured VM's
  - Client-01 and DC-1 Virtual Machines
  - MircoSoft Active Directory installed in DC-1
  - DC-1 promoted to a Domain Controller
  

![Screen Shot 2023-12-23 at 4 15 12 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/c269c7e1-0b49-4090-b4d2-d69d95d68b08)


<h2>Understanding DNS</h2>

- DNS converts computer names (such as client-1.mydomain.com or www.google.com) to IP addresses which can be used by the computer to locate resources
- DNS is integrated with Active Directory and got automatically installed on DC-1 in my guide prior to this (https://github.com/Emq17/Configuring-On-premises-Active-Directory-within-Azure-VMs)
- In this lab, we will use Client-1 and DC-1 to do a few exercises for the sake of understanding DNS a bit better

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

<h2>CNAME Records</h2>

Canonical Name; A DNS CNAME record provides an alias for another domain.

- First go to Client-1 and ping "search" and see that nothing was found

![Screen Shot 2023-12-23 at 10 26 40 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/f089fb5b-2f7f-4c7f-8aea-9fa15e03e8d2)

- Go back to DC-1's DNS Manager and create a new CNAME record
  - Right click, choose "New Alias (CNAME)..."
 
![Screen Shot 2023-12-23 at 10 29 20 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/bd721ac8-cc9d-407b-8b12-28377a95d72b)

- Point the host "search" to associate with "www.google.com"
  - We are just trying to show that we can map names to other names

![Screen Shot 2023-12-23 at 10 30 06 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/cf055bfd-0c3e-4674-ab8c-abcbd4bef93c)
![Screen Shot 2023-12-23 at 10 34 22 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/efc160c4-a0be-43c8-8e7e-77d217bb4fac)

- Now go back to Client-1 and ping "search" once again like last time
  - Notice how it resolves to "www.google.com"

![Screen Shot 2023-12-23 at 10 35 44 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/a85dd2f1-ac8d-4a77-bbf6-a0b5db04b0ec)

- Just for fun, open up your browser and type in your "fully qualified domain name"

![Screen Shot 2023-12-23 at 10 37 28 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/5c62318f-a7b1-4c15-b7b9-d03f1711aa36)

- Notice that it tried to load www.google.com but the certificate did not match so it threw an error code
  - This is probably going beyond the scope of this demonstration but still it is pretty cool how we forced the name "search" to resolve to "www.google.com" through CNAME Records

![Screen Shot 2023-12-23 at 10 40 06 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/b16e0d4b-e2e6-48d3-8cbd-37e1379954f4)

- By using "ipconfig /displaydns" you can visually see how "search.mydomain.com" resolves to "www.google.com" and then how "www.google.com" resolves to its IP address

![Screen Shot 2023-12-23 at 10 43 55 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/60d507ac-75af-430f-84ef-481fe373aca4)

<h2>Root Hints</h2>

The authoritative name servers that serve the DNS root zone, commonly known as the "root servers", are a network of hundreds of servers in many countries around the world. They are configured in the DNS root zone as 13 named authorities; Root hints resolve queries for zones that do not exist on the local DNS server. They are only used if forwarders are not configured or fail to respond.

- By going to DC-1's DNS manager, right clicking "DC-1", choosing "Properties", and opening up the "Root Hints" tab, you can see all the servers 

![Screen Shot 2023-12-23 at 11 25 45 PM](https://github.com/Emq17/Building-Intuition-For-DNS/assets/147126755/f9f156e4-c07e-42e5-aa75-93d865e2e405)

To summarize, root hints are a set of IP addresses pointing to the root DNS servers, and they help DNS resolvers initiate the process of resolving domain names by providing the initial information needed to navigate the hierarchical structure of the DNS. Once the resolver has the information from the root hints, it can proceed with queries to the appropriate authoritative DNS servers to ultimately resolve the requested domain name.

<h1>Congratulations on finishing another walkthrough with me</h1>

I hope these step-by-step visuals made the topics we've discussed above clear enough for you to get a good handle on how DNS overall works
