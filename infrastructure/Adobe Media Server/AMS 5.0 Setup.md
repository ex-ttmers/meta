### Setup

The AMS uses the following AWS services:
  1. Adobe Media Server 5.0 AMI Based on CentOS
  2. ELB - Provieds a Static IP
  3. AutoScaling Group
  4. Autoscaling Launch Configuration

To create the TTM AMI
  1. Launch an instance of the AMS AMI from the market place
  2. Install the TTM Action Script files and configure the server. See the PZAudioCZ GitHub Repository.
  3. Take an image of the configured server.

Create an instance of an ELB. Forward port TCP port 1935 to the host.
Create a securty group to pass through TCP/UDP 1935 and TCP 22 for SSH
Create a SSH key pair for use with the servers spawned by the autoscaler

We use the autoscaler to shut down the AMS outside of live teaching hours. To configure the autoscalr:
  1. Create an Autoscaling Launch Configuration using the TTM AMI. Us the security group created above
  2. Create an Autoscaling group using the above launch configuration and associate it with the ELB created above.

To configure date time based scaling, use the aws command line. See http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/schedule_time.html

Remember times are scheduled in UTC.

### Accessing

The PZ and CZ flash apps require port 1935.
If you opened ssh that runs on port 22

To gain acess to the AMS Web GUI you need port 1111 and 80. I suggest using an SSH tunnel so that those ports are not opened to the world.
