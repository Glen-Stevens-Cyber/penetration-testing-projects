# Web Application Security Testing

## Project overview

This Netlab activity focuses on web penetration and uses two well-known web application assessment tools for conducting security assessments. This activity took place in an isolated environment containing virtual machines which were configured for cybersecurity training during my BSc(Honours) Cyber Security studies at the Open University.

The project introduced a practical web security testing workflow using Nikto and Burp Suite Community Edition. I used Nikto to scan a deliberately vulnerable web server and review the findings inside of a report generated to be viewed in HTML. I then configured Firefox to send web traffic through my device’s loopback interface so it could be captured in Burp Suite, which allowed me to map the target application, examine a captured login request and use Burp Intruder to test how the application responded to multiple authentication attempts.

This activity was a guided security-testing exercise rather than a complete professional penetration test. It focused on building familiarity with vulnerability scanning, HTTP request analysis, proxy configuration, application mapping and authentication testing.

## Activity objectives

The main objectives were to:

- Use Nikto to scan a deliberately vulnerable web server.
- Review security observations reported by Nikto.
- Export and examine scan findings in an HTML report.
- Configure Firefox's proxy settings to route web traffic through Burp Suite.
- Use Burp Suite to map requested application resources.
- Examine HTTP requests, parameters, cookies and responses.
- Configure Burp Intruder payload positions.
- Supply possible password values from a wordlist.
- Use response matching from previous failed login attempts to distinguish failed authentication attempts.
- Identify an anomalous response requiring further investigation.
- Document the work without exposing credentials or session identifiers.

## Tools and technologies

- **Kali Linux:** The testing workstation used to run Nikto, Firefox and Burp Suite.
- **Nikto:** A web-server scanner used to perform fast automated checks on web servers to identify potential vulnerabilities like out-of-date software and vulnerabilities in plugins.
- **Burp Suite Community Edition:** A web security testing platform used with a proxy that is used to sit between a browser and a server to capture, intercept and modify web traffic.
- **Firefox:** Browser that was used and configured with a proxy configuration, so browser traffic was sent through Burp Suite's local proxy listener. 
- **OWASP Broken Web Applications:** The deliberately vulnerable target system used inside the lab.
- **Damn Vulnerable Web Application (DVWA):** The target web application used to examine login requests and test authentication behaviour.
- **HTTP:** The application-layer protocol examined throughout the activity.
- **HTML:** Used by Nikto to present the exported scan report.
- **Password wordlist:** A prepared collection of possible password values supplied to Burp Intruder during the authorised test.

## Authorised NetLab topology

Image: '1. Topology - Copy.png'

The topology shows the isolated NetLab environment used during the activity. It contained several virtual machines connected across LAN, WAN and DMZ networks through pfSense. Kali Linux acted as the testing workstation, while the OWASP Broken Web Applications server at `192.168.68.12` provided the deliberately vulnerable target.

## Nikto vulnerability scanning

Image: '2. Nikto Terminal Scan - Copy.png'

I began the first part of the activity on the Kali machine and opened a terminal. Inside of that terminal I began by entering the following command to see what type of scans I could perform with Nikto:

**nikto -help** 

Once I evaluated my options, I then used the following command to perform a general scan on the web server OWASP in this activity:

- **nikto -Plugins tests -host 192.168.68.12**

The command can be broken down as follows:

- **`nikto`:** Starts the Nikto web-server scanner which will look to identify potential vulnerabilities like out-of-date software or other security issues for example.
- **`-Plugins tests`:** Selects Nikto’s built-in tests plugin, which performs checks for potentially dangerous or interesting files, directories and web-server configurations.
- **`-host 192.168.68.12`:** This part specifies that we want Nikto to target our OWASP Broken Web Applications server for the scan.

The image highlights how Nikto performed the scan and completed 24,629 requests. The scan provided general feedback inside of the terminal and reported 8 items which will be expanded and touched on further in the next section.

## Nikto HTML report

Image: '3. Nikto HTML Report - Copy.png'

This part carries on from the previous section. To be able to review the findings of the scan completed previously in a more readable format and to read the descriptions associated with each finding, I used the following command to repeat the scan again but this time, export my findings into a file to be viewed in HTML: 

- **nikto -host 192.168.68.12 -output report.html**

The HTML report provided a structured view of Nikto's findings. It identified the target web server, port number and software versions before listing several possible security concerns. An example would be at the top of the report where Nikto identified the web-server technologies and their reported version numbers. This is not a vulnerability but having unnecessary information publicly retrievable when it doesn't need to be can lead to reconnaissance being made easier for the attacker as certain version numbers will have certain weaknesses associated with that version. This also enforces the need for regular patching needing to be done for all software associated with the web server.

## Browser proxy configuration

Image: '4. Proxy Configuration - Copy.png'

Before moving on to using Burp Suite, I needed to go into Firefox's preference and network settings to apply the following configurations for the upcoming activities:

- **127.0.0.1:** This is the loopback address, and I entered to ensure that all browser traffic I create for the next few activities is directed back to my own system to capture. This allowed for the traffic to go from Firefox to my Burp Suite application and then on to the destination address which highlights the importance of the proxy being placed between the two.
- **Port 8080:** The local port used by the Burp Suite proxy listener.

## Application mapping and captured login request

Image: '5. Burp Suite Captured Login Request + Site Map - Copy.png'

This part of activity was the start of me working with Burp Suite. To start this part of the activity, I opened Firefox and navigated to the broken web app's IP address by typing 192.168.68.12. Once I searched for this, I immediately returned to Burp Suite and verified that this traffic was captured and when I looked inside, I was able to verify that this same IP address was located in the site map under the targets tab.

To ensure the focus was placed on this IP address alone, I added this address to the scope for filtering, and this allows for Burp Suite to place an emphasis on this IP address and disables out of scope logging.

I then navigated back to the targets tab, then to the site map and expanded the IP address located on the left which allowed me to see all the directories on the website. As I browsed through the OWASP, Burp Suite populated its Target site map with directories, files and parameterised requests discovered during the session. Entries displayed in black represent resources that were actually requested, while grey entries represent resources that Burp discovered but which had not yet been requested. Because I had only just reached the IP address, the majority of the entries were in grey. 

I returned to Firefox and clicked on the 'Damn Vulnerable Web Application' (DVWA) which is where it prompted me for login details. I submitted the correct login details which were supplied to me for this activity by the university. Once submitted, I returned to the Burp Suite, and I noticed that more entries on the left had turned black because I navigated to them which allowed Burp Suite to discover more. After navigating to DVWA and entering my login details there on the page, that folder labelled 'dvwa' had turned black, so I expanded that folder and saw that Burp Suite had also captured my login details that I submitted.

This is what is captured in the screenshot located in my repository. The screenshot shows that Burp Suite was able to capture the POST request which came from Firefox containing my login details which have now been added to the target site map. When clicking on my login details, on the right side of the page I clicked on Params below the request tab and it contained the type, the name and the values which have been blurred out which were submitted in the POST request.

Credential and session values have been redacted from the public screenshot. The parameter names and request details remain visible to demonstrate the analysis without publishing reusable authentication information.

## Burp Intruder payload positions

Image: '6. Burp Suite Intruder Payload Positions - Copy.png'

For this part of the activity, we will be conducting a brute force attack on the OWASP broken web application. 

First step was to navigate to a different part of the webpage which also required a set of login details which we don't have the correct details for which presents the need to perform an authorised brute force attack. 

Once I submitted a set of login details which were incorrect, I went back to Burp Suite to find that GET request I just tried to submit which contains the login details. Once found, I sent this to the Intruder tab which is what we'll be using to perform the brute force attack because of its abilities to submit automated requests while also changing the values automatically and observing each response. 

I sent the authentication request to Burp Intruder and selected the username and password values as payload positions. Payload positions tell Intruder which parts of the request should be replaced during each test. 

To select the username and password as values for the payload positions to be replaced during each submission, we need to contain them inside of the section signs, one at the beginning and one at the end of each section. 

The activity used the **Cluster bomb** attack type. This mode uses a separate payload set for each selected position and tests combinations of the supplied values. In this request, one position represented the username and the other represented the password.

## Password payload configuration

Image: '7. Intruder Password Payload Configuration - Copy.png'

This image highlights the payload that we will be altering with each login attempt of the brute force attack. We will be altering the second payload referring to the position of the password. We have only provided one value for the first payload set which refers to the username and that was the value we provided in our previous login attempt which won't be disclosed.

I have also selected the **Runtime File** as the payload type which will go through the 492 options of passwords we provide in the file below. This means that the cluster bomb attack will use the one username we provided and combine that with all of the 492 options contained in the runtime file to find the correct password. This method highlights the benefit in using Intruder and Burp Suite because they can automate this process for me instead of me manually having to go through every entry.

## Identifying failed responses with Grep-Match

Image: '8. Grep-Match Configuration - Copy.png'

For this part of the Burp Suite configuration, we moved to the options tab and needed to add a rule to our attack to identify when our submission failed. This is where the Grep-Match is useful and to make full use of this I needed to open Firefox again and analyse what is displayed when the login details are incorrect. The login screen displayed the following:

'Username and/or password is incorrect'

Going back to the Grep-match section, I typed the word 'incorrect' and added this rule to the attack configuration which means that all attempts that are responded to with the same phrase which contains the word 'incorrect' will be marked and the one that isn't incorrect won't be marked so I’ll be able to spot the anomaly. 

## Intruder results analysis

Image: '9. Burp Suite Intruder Results.png'

Once the Grep-match was configured, the cluster bomb attack was initiated, and the screenshot highlights the intruder results.

Inside of the screenshot, the payload values have been redacted from the screenshot so that the public repository does not disclose laboratory credentials or supplied answers. Despite the values not being visible in the screenshot, we can notice that request 1 is different from the other requests because of the following reasons:

- Response length was 5283 and not 5222 like the rest of the values
- The Grep-Match indicator for `incorrect` was not selected.

These two differences allowed me to spot the anomaly and allow me to assume that this authentication attempt was successful. However, these two differences alone wouldn't be enough to trust that the payload values would grant me access to the page were trying to access via brute force. I can only be sure by entering the details myself so I can verify if the anomaly value is correct.

## Skills demonstrated

This project demonstrates experience with:

- Running and interpreting Nikto web-server scans.
- Exporting and reviewing HTML vulnerability reports.
- Configuring a browser to use a local intercepting proxy.
- Understanding loopback addressing and proxy ports.
- Mapping web application resources with Burp Suite.
- Examining HTTP methods, status codes, cookies and parameters.
- Recognising the importance of protecting session identifiers.
- Configuring Burp Intruder payload positions and attack types.
- Supplying password candidates from a runtime wordlist.
- Using Grep-Match to classify responses.
- Comparing response content and length rather than relying only on status codes.
- Distinguishing a suspicious result from a confirmed finding.
- Redacting credentials and session data before publishing evidence.

## What I learned

This activity gave me a great introduction to using useful security tools like Nikto, Burp Suite and also taught me how to configure my own proxy in a browser to be used alongside Burp Suite. Using Nikto allowed me to see what a valuable tool it can be when pen testing by its ability to quickly identify files, directories, software information and configuration observations. Converting this information into a report provided a format that improved readability and made it even easier to understand the observations it was able to detect.

Allowing me to learn how a proxy sits between a browser and a web application allowed me to understand the benefit and the use in configuring a proxy on Firefox. Configuring the proxy with the loopback interface so the traffic would have to go through my device which allows Burp Suite to capture the HTTP/S traffic generated through my device.  

My experience of using Burp Intruder was valuable because it was a brilliant example of how an automated brute force attack could happen by altering the payloads involved using repetitive tests and Grep-Match being able to classify responses by detecting the word specified before the attack started. I also learned that identical status codes in the results of the intruder attack do not necessarily mean identical outcomes which we saw with the anomaly that also had the same status code of 200 which was the same as the other attempts. The differing response length and missing failure string provided evidence of an anomaly, but further verification would still be necessary before describing it as a confirmed successful login. 

## Ethical and repository note

All scanning, proxying and authentication testing documented in this project took place inside an isolated and authorised Open University NetLab environment containing deliberately vulnerable training systems. No public, production or unauthorised systems were targeted.

The original university activity instructions, supplied files, credentials, session identifiers and complete wordlists have not been included in this public repository. Sensitive values have been redacted from the screenshots. The evidence is provided to demonstrate the testing process and the skills developed without publishing university answers or reusable authentication information.

