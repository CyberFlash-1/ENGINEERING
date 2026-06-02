
Phase 1 deployment
 <img width="975" height="792" alt="image" src="https://github.com/user-attachments/assets/3f5e3df7-0f97-42cf-9474-ed6361e8411d" />

Phase one of the deployment focuses on getting all of the data in to splunk.
•	Linux UF’s installed on all syslog servers (Boston,London,Ashburn,San Francisco)
•	The network devices continue to send their logs to their local syslog server
•	Windows UF on SEPM, SEPM aggregates all endpoint security logs


 <img width="975" height="671" alt="image" src="https://github.com/user-attachments/assets/eaa652ea-29f2-498f-8235-06a7dbd8c3f2" />

In phase 2: the management tier is included (CM, LM, MC, DS)
•	All syslog servers have a Linux UF for sending data to Splunk 
•	All ecommerce servers have moved directly to include a Universal Forwarder- for reliability and reducing the syslog load
•	Additionally, the add-on of cisco networks app has been introduced 
o	Why: This app best helps achieve the customers priority which was to ensure the integrity and availability of ecommerce data
o	The app directly supports these goals by providing visibility into ACL denies, routing changes, interface errors, early warnings for outages
