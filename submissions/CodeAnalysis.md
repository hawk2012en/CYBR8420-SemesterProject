## Code Review Strategy

Before diving into the code analysis, we started with reviewing our assurance cases, misuse cases, and threat model report. From the review, we observed the most critical weakness areas in the software and decided to approach the checklist review strategy which is would most appropriate method for analyzing the code of large-scale projects like Elasticsearch. To reduce the effort of going through all the code for each line we decided on adopting to review all items from our checklist.

We analyzed our code in both automated and manual review process. The automated code review is performed using open source static code analysis tool called **PWD and Findbugs**. The list of resultant vulnerabilities from the automated tool scan was narrowed  down to analyze  **CWE(Common Weakness Enumeration)** that are related to our misuse case, assurance cases, and threat model and examined manually. 

## Manual Code Review

From our misuse case [User updates data](https://github.com/swrp/CYBR8420-SemesterProject/blob/master/submissions/SoftwareSecurityRequirements.md#User-updates-data) and threat report, we identified vulnerability related to [CWE-798: Use of Hard-coded Credentials](https://cwe.mitre.org/data/definitions/798.html). There are maximum chances of attacker finding a loop hole from this vulnerability that can lead to the disclosure of sensitive information and gaining unauthorized access to sensitive files. The code issue is located in the file [KeyStoreWrapper at line 40]( https://github.com/elastic/elasticsearch/blob/master/x-pack/plugin/core/src/main/java/org/elasticsearch/license/CryptUtils.java). 
At the line 40 default pass phrase key value is **hard coded** which is used to decrypt the private key file. Knowing the public key, private key and algorithm methods attacker can able to find the back door to break the algorithm patterns used.

Using our misuse case [User updates data](https://github.com/swrp/CYBR8420-SemesterProject/blob/master/submissions/SoftwareSecurityRequirements.md#User-updates-data) and threat report, we performed manual code review to verify the [CWE-284: Improper Access Control]( https://cwe.mitre.org/data/definitions/284.html). We have observed that all security implementations were accomplished by managing the settings for **Access Control based on user role** and handling document-level privileges. There is medium severity issue related to the **Trusting all the incoming SSL certificates** from the server. This is most relatable to [CWE-300: Channel Accessible by Non-Endpoint](https://cwe.mitre.org/data/definitions/300.html) which could potentially maximize the chances of affecting the integrity and confidentiality of the data. Attacker can read or modify the communication taking advantage of this weakness. 
However, Elasticsearch implemented the mitigation to this vulnerability by verfying the certicates using standrd X.509 at [Line 91](https://github.com/elastic/elasticsearch/blob/e179fd1274331e30bc3ddfb1ab789fb0d38c92bd/x-pack/plugin/core/src/main/java/org/elasticsearch/xpack/core/ssl/RestrictedTrustManager.java#L91) in RestrictedTrustManager.java.

For the use case [System Uploads Data](https://github.com/swrp/CYBR8420-SemesterProject/blob/master/submissions/SoftwareSecurityRequirements.md), input sanitization was identified as the most important security requirement. The manual code analysis was hence focussed on th eCWEs that challenge input sanitization. The following CWEs were identified and investigated:

[CWE-502: Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html)
This medium-severity vulnerability has been mitigated in many portions of the code base, however in Line 117 of [DataHistogramGroupConfig.java](https://github.com/elastic/elasticsearch/blob/master/x-pack/plugin/core/src/main/java/org/elasticsearch/xpack/core/rollup/job/DateHistogramGroupConfig.java) the vulnerability is still present.
There is no way of knowing what is being deserialized before it has been decoded. This flaw could be exploitaed to effect a complete remote command execution and the host could takeover on the application accepting serialized objects.

**Mitigation:** The return type and the list of the expected classes that might be accepted to show up in serialized objects should be specified. Also, there can be a limit enforced on the number of the embedded objects and the bytes of input.

[CWE-22: Improper Limitation of a Pathname to a restricted Directory ('Path Traversal')](http://cwe.mitre.org/data/definitions/22.html)
This is one of the SANS Top-25 and could lead to code-execution, data loss and denial of service when an outsider gives an input while constructing a filename and the resulting path points  outside of the intended directory. This vulnerability has been well remidiated in Elasticsearch as they decode and canonicalize inputs to the application current internal representation before validating them. The data validation can be viewed in the [FileUtils.java](https://github.com/elastic/elasticsearch/blob/master/qa/vagrant/src/main/java/org/elasticsearch/packaging/util/FileUtils.java)






## Automated Tool Scan 
