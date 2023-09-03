# General system health check monitor

### High-level proposal
The purpose of this feature is to provide a way for the user to monitor the overall health of their AmazingOS system, such as disk usage, network connectivity, kernel errors, etc. 

- The feature will consist of a C++ daemon that will run periodically (e.g. every 10 minutes) and execute various Linux commands using the Boost.Process library to collect system information. 
- The daemon will also connect to a PostgreSQL database to store the collected data and generate reports. 
- The daemon will communicate with an AWS service (e.g. CloudWatch) to send alerts if any system issue is detected.

### Resources 

Some packages that could be used for this feature are:

- Boost.Process: A C++ library that provides a cross-platform way to execute and communicate with child processes. It can be used to run Linux commands and capture their output.
- PostgreSQL: A relational database system that can store and query the system health data.
- AWS CloudWatch: A cloud service that can monitor and alert on various metrics and events. It can be used to send notifications if any system issue is detected.

Some articles online that could help the developer understand the concepts are:

- [How to Monitor Linux Systems Performance with iostat command](https://www.linuxtechi.com/monitor-linux-systems-performance-iostat-command/)
- [7 Linux Commands to Check Network Connectivity](https://www.comptia.org/blog/7-linux-commands-to-check-network-connectivity)
- [Use dmseg to check your Linux system's kernel message buffer](https://www.networkworld.com/article/3643696/use-dmseg-to-check-your-linux-systems-kernel-message-buffer.html)
- [Boost.Process documentation](https://www.boost.org/doc/libs/1_64_0/doc/html/process.html)
- [Connecting to PostgreSQL with C++](https://www.instaclustr.com/support/documentation/postgresql/using-postgresql/connecting-to-postgresql-with-c-plus-plus/#:~:text=The%20easiest%20way%20to%20set,official%20documentation%20for%20more%20information.)
- [Monitor and trigger alerts using Amazon CloudWatch for Amazon Connect](https://aws.amazon.com/pt/blogs/contact-center/monitor-and-trigger-alerts-using-amazon-cloudwatch-for-amazon-connect/)

### Open Questions 

Questions that need to be cleared up with the stakeholders:

- What are the specific metrics and thresholds that define a healthy system?
- How often should the system health check run and how long should the data be stored?
- What are the preferred channels and formats for sending alerts?
- How should the system health reports be presented and accessed by the user?

Possible risks:

- The system health check daemon may consume too much CPU or memory resources while running.
- The system health data may grow too large and require more disk space or database maintenance.
- The communication with AWS CloudWatch may fail or incur additional costs.