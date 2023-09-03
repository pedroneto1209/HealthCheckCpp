# Question 1

While developing sophisticated software to increase motorcycle rider safety, which involved dealing with RADAR, cameras, and complex components, I encountered a problem that can be both simple and complex at the same time. What makes me proud of this code in particular is that I had to deal into low-level programming to reimplement two much used memory allocation functions: `malloc()` and `free()`. When you are accustomed to using them every day, it's easy to underestimate the beauty and complexity of some fundamental programming concepts. 

The incident that occurred was that we were experiencing issues with functions utilizing malloc for memory allocation, and through debugging, we identified that this function's output was misleading and affecting our heap in unintended ways.

The solution I came up with was to reimplement `malloc()` and `free()` on my own, and although it would be simpler than our supplier's implementation, it would get the job done as expected and not mess with wrong heap areas. This is part of the reimplementation with comments:

NOTE: The heap was managed using a linked list, and it was already provided by the supplier after the MCU initialization.

## Code


```cpp
// This will use the "best fit" approach to find the best memory block to alloc,
// it will consider the shortest block that can contain the size defined
Block_t *find_best_block(Block_t *current, size_t size) {

    Block_t *min_size_block = nullptr;
    size_t min_size = 128;

    while (current->size < 129) {
        if (current->free && current->size > size) {

            if (current->size == size) {
                std::cout << "Best block is: " << current->size
                          << ", Next: " << current->next
                          << ", Previous: " << current->previous
                          << ", Free: " << current->free << std::endl;

                return current;
            }

            if (current->size < min_size) {
                min_size = current->size;
                min_size_block = current;
            }
        }
        current = current->next;
    }

    if (min_size_block != nullptr) {
        std::cout << "Best block is: " << min_size_block->size
                  << ", Next: " << min_size_block->next
                  << ", Previous: " << min_size_block->previous
                  << ", Free: " << min_size_block->free << std::endl;
    }

    return min_size_block;
}

// This will allocate the passed block and create another block
// if there is any space left
void *alloc_block(Block_t *best_block, size_t size) {
    void *ptr;

    if (best_block->size == size) {
        best_block->free = 0;
        ptr = &best_block;
    } else {
        Block_t *new_block = new Block_t;

        new_block->size = best_block->size - size;
        new_block->free = 1;
        new_block->next = best_block->next;
        new_block->previous = best_block;

        best_block->size = size;
        best_block->free = 0;
        best_block->next = new_block;

        ptr = &best_block;
    }

    return ptr;
}

// This will run all the functions after making size checks
void *malloc_redefine(size_t size) {
    std::cout << "malloc_redefine started" << std::endl;

    if (size > HEAP_SIZE_MAX || size == 0) {
        return NULL;
    }

    // this function returns the heap linked list already defined by the MCU
    heap_head = get_heap();

    // this is a helper to print the heap
    print_heap(heap_head);

    Block_t *best_block = find_best_block(heap_head, size);

    alloc_block(best_block, size);

    print_heap(heap_head);

    return best_block;
}

// This function sets the block allocated to free, also, the adjacents blocks
// are checked, if they are free this function will join into a bigger block,
// resulting in less fragmented memmory
void free_redefine(void *ptr) {
    std::cout << "free_redefine started" << std::endl;
    Block_t *block = (Block_t *)ptr;

    std::cout << block->next << std::endl;

    block->free = 1;

    if (block->previous->free) {
        block->previous->previous->next = block;
        block->previous = block->previous->previous;
        block->size += block->previous->size;
    }

    if (block->next->free) {
        block->next->next->previous = block;
        block->next = block->next->next;
        block->size += block->next->size;
    }

    print_heap(heap_head);
}
```


# Question 2

## Daily Brief Security Report

Date: September 2, 2023

Dear Software Engineering Manager,

I hope this message finds you well. As the head of the cybersecurity department for AmazingOS, I wanted to provide you with an urgent security update regarding two newly discovered vulnerabilities that affect our operating system, AmazingOS.

### CVE-2022-48303

CVE-2022-48303 represents a security vulnerability characterized by a one-byte out-of-bounds read within GNU Tar. This flaw leads to the utilization of uninitialized memory for a conditional jump. The affected software includes GNU Tar versions up to and including 1.34, which is utilized by the AmazingOS platform. 

The vulnerability manifests in the 'from_header' function within the 'list.c' module via a V7 archive in which mtime has approximately 11 whitespace characters. Its severity is rated as medium, with a CVSS score of 5.5, and it has the potential to enable an attacker to initiate a denial of service attack or potentially execute arbitrary code.

### Actions to take

- Apply the patch provided by GNU Tar developers as soon as possible or update the GNU Tar version to 1.35 or later. Alternatively, you can avoid opening V7 files with whitespace characters in the mtime parameter.

### CVE-2021-44228

CVE-2021-44228 is a remote code execution vulnerability in Apache Log4j2 that allows an attacker to execute arbitrary code loaded from LDAP servers when message lookup substitution is enabled. It affects Apache Log4j2 2.0-beta9 through 2.15.0 (excluding security releases 2.12.2, 2.12.3, and 2.3.1), which are used by the WebSphere Application Server Admin Console and the UDDI Registry Application in AmazingOS.

`The severity of this vulnerability is critical (CVSS score 10) and it could allow an attacker to take complete control of the system.` This vulnerability is also called Log4Shell or LogJam.

### Actions to take

- Upgrade to a newer version of Apache Log4j2 that is not affected by this vulnerability (2.16.0 or later). 
- Alternatively, set the JVM custom property log4j2.formatMsgNoLookups to the value `true`.
- We recommend to update the IBM SDK, Java Technology Edition maintenance to the latest recommended fix pack.
- If you are using WebSphere Application Server, you should apply the interim fix PH42728 as soon as possible and redeploy the UDDI Registry Application if it is running on your server.



We are very proud to our engineers for the rapid follow up about the problem and working closely through its resolution. All of which occurred without any reports that outside parties had identified the issue or exploited it.

Furthermore, we recommend to take actions on notifying the customers about these vulnerabilities and provide them with the fixes described until the next release of AmazingOS is available.

Also, be aware to monitor the network traffic and logs for any signs of exploitation attempts or malicious activity related to these vulnerabilities.

Please let me know if you have any questions or require further information.

Best regards,

Pedro Neto<br>
Head of Cybersecurity Department<br>
AmazingOS Company


# Question 3

## Software Implementation Proposal: General system health check monitor

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