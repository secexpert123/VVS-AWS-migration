# Evidence

Screenshots from my own AWS Learner Lab test (Lab 1). Account ID, email and my own IP addresses are blurred.

1. **1-ec2-running.png:** my EC2 instance `lab1-web-as` (t3.micro, Amazon Linux 2023) in state Running.
2. **2-security-group-ssh-my-ip.png:** the security group after I restricted SSH (port 22) to my own IP. HTTP (port 80) stays open to everyone.
3. **3-ssh-log-check.png:** the SSH log check I ran before restricting port 22. The search for "invalid user" gave 0 results, but one outside IP connected to port 22 and closed the connection before login. The IPs in the log are AWS's connect tool and an outside scanner, not mine.
4. **4-s3-bucket-with-file.png:** my S3 bucket with the uploaded `index.html`.
