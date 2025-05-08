A REPORT
 
ON
 
WEB ACCESSBILITY DETECTOR AND ENHANCER
BY
 
Mamidisetty Keshava Teja                                                                           ID.No:2023HT66014
AT  
Hyderabad
 
Copart India Center, Hyderabad
 
  
BIRLA INSTITUTE OF TECHNOLOGY & SCIENCE, PILANI 
 
(April, 2025)



A REPORT
 
ON
 
WEB ACCESSBILITY DETECTOR AND ENHANCER
 
BY
 
 
    	Mamidisetty Keshava Teja                 	                             ID.No. : 2023HT66014	        


 
Prepared in partial fulfilment of the
WILP Dissertation/Project/Project Work Course
AT 
 
Copart India Center, Hyderabad




Acknowledgements:

I sincerely thank each one who are in this part of my journey.

·         Niranjan Josyula, VP and Director, CIC and Copart for giving the opportunity to the WILP course.
·         Suresh Kusuma, Tech lead, Devops Engineering, Copart.
·         Santosh Samineni, Teah Lead, Site reliability Engineer, Copart.
·         K Hari Babu, Faculty mentor. BITS pilani, WILP.
.         Stackstorm slack community who answered questions while setting up stackstorm.

















Introduction: 
To add components of the web application in new relic and using the newrelic platform to integrate and organize web application's user interface components, such as charts and grids, to enhance functionality and readability. Once integrated, monitoring these components using New Relic's Application Performance Monitoring (APM) tools, which provide insights into performance metrics, user experience, and potential issues across the application stack. For auto-remediation, Stackstorm, as an event-driven automation platform to detect and resolve issues automatically. 
By setting up diagnostic and remediation workflows in Stackstorm, automation of  tasks like restarting services, clearing disk space, or addressing performance bottlenecks, ensuring minimal downtime and efficient issue resolution are achieved. 
Combining New Relic's monitoring capabilities with Stackstorm's automation creates a robust system for maintaining and optimizing web applications.
The Project key areas of work would be DevOps, Networking and systems design architecture.

Table of Contents
Components in web application	1
Adding applications in new relic	2 and 3
Setting up Stackstorm	4
Newrelic – Stackstorm integration	5
Domain and network setup	6
Stackstorm configuration	7-12
Conclusions and Recommendations…………………………………………………………………...13
References…………………………………………………………………………….14




 

1. Components in Web application 
Web application consists of following components. 
1.Frontend 
2.MongoDB 
3.Catalogue 
4.Redis 
5.User 
6.Cart 
7.MySQL 
8.Shipping 
9.RabbitMQ 
10.Payment 
11.Dispatch
All these web applications are deployed using virtual machines, setup was done using rocky linux vms in esxi platform. here newrelic vm is a private component which reports local data which can be accessed only in local environment from these hosts to newrelic. 


Adding applications in newrelic application: 
 

All these applications are added to new relic, (a free newrelic account signup has been created using google account). 
Newrelic agents have been deployed to each server and now the hosts are reporting to Newrelic.
 
alert conditions have been setup when services go down, high CPU, disk and memory usage, 
Thresholds will be mentioned as per requirement in alert condition
 
Alert condition query in the screenshot
 
Setting up Stackstorm: 
Stackstorm configuration: 
The server was setup using custom username and password for Stackstorm using the script – 
bash <(curl -sSL https://stackstorm.com/packages/install.sh) --user=st2admin --password=Ch@ngeMe 
ref.  https://docs.stackstorm.com/install/index.html#ref-one-line-install  
this script installs all the components and pre-requisites that stackstorm needs 
mongodb – for database and default username and password is set as is.
Rabbitmq messaging queue as a message broker to enable communication between various components.
A username is configured for rabbitmq user with below command
sudo rabbitmqctl add_user <username> <password>
RabbitMQ username and password was configured in /etc/st2/st2.conf file 
these steps are crucial to ensure that stackstorm user interface is up and running.
Newrelic – stackstorm integration:
Stackstorm has a buildin triggers /api/v1/webhooks endpoint.
this webhook endpoint api need to be configured in newrelic so that it can send JSON payload to stackstorm, stackstorm then uses the data from the payload and perform the defined action we set in actions following rules.
But Newrelic webhooks only works when the api URL is secured with SSL.
To make URL secure a domain keshavlab.org was purchased via cloudflare vendor and dns is configured and ssl certificates downloaded and installed in stackstorm nginx configuration.
 

an A record is configured in DNS configuration for stackstrom which is st2 and web URL for stackstorm is https:st2.keshavlab.org
 

PORT forwarding: created port forwarding rules to divert traffic from different internal applications to make public, for example making stackstorm api URL to make public.
 

Origin server SSL certificates are generated in Cloudflare and installed in nginx configuration of stackstorm 
 
Nginx configuration inside stackstrom server.
 

now the api URL is secure and can add to newrelic. https://st2.keshavlab.org/api/v1/webhooks/newrelic 
Api key and value has been generated in stackstorm server using command:
st2 apikey create -k -m '{"used_by": "my integration"}'   ---reference docs: https://docs.stackstorm.com/authentication.html 
A webhook destination is configured in newrelic using above stackstorm api

 

When any new relic alert is triggered, the alert will send notification message which is JSON payload to stackstorm.
Payload Preview in newrelic:
 
 
A Newrelic pack has been created to restart nginx /opt/stackstorm/packs/newrelic_nginx, this directory should contain action, metadata, pack.yaml and rules,ref: https://docs.stackstorm.com/reference/packs.html 
 
Under actions the definition of the action that need to performed is configured, here in this case a simple nginx service restart.
 
Under the rules directory a yaml file has been created, here the criteria is defined, when there’s criteria match from the trigger’s payload, stackstorm runs the action defined in the rule – newrelic nginx.restart.
 

to test manually made nginx down, and the stackstorm performed auto restart and the website is up with very minimal downtime.
 
 
Frontend is up.
 











Conclusions and Recommendations:
Thus, the Web accessibility detector and enhancer is achieved for smooth running of website using DevOps tools new relic and stackstorm.
Stackstorm can be easily implemented once the reference documents in the stackstorm website are fully understood and knowing purpose of each component like packs, sensors and webhooks.
This project consists of several key areas like networking concepts, domain, how to setup domain, securing a website using SSL, newrelic and stackstorm automation. A thorough understanding of infra level design and architecture, monitoring and auto remediation can be achieved.














References: 
https://docs.stackstorm.com/webhooks.html 
https://docs.stackstorm.com/rules.html 
https://docs.stackstorm.com/authentication.html 


Glossary:
Webhook: A webhook is a push-based event notifier to send real time data using HTTP post request.
Packs: A stackstrom pack consists of actions and rules and definitions of how to perform an action using rules after receiving JSON payload from newrelic. 
