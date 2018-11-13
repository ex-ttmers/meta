## Configuring ELB for WebSockets

ELB does not support WebSockets when using HTTP and HTTPS listners. In order for WebSockets to work,
ELB must be reconfigured in the following way.

    * Change the listner protocol to TCP and or SSL instead of HTTP/HTTPS
    * Attach a proxy protocol policy to the ELB
    * Configure back end servers to use the proxy protocol header
    * Update or Configure ELB health check to use the TCP/SSL protocol and not HTTP/HTTPS
    

### Attaching a proxy protocol policy
    * See http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-proxy-protocol.html

#### Steps to configure a farm

    1. Create a proxy protocol policy for the load blancer. 
        `aws elb create-load-balancer-policy --load-balancer-name <LOAD BALANCER NAME FROM AWS OR SCALR> \
         --policy-name <POLICY NAME> --policy-type-name ProxyProtocolPolicyType \
         --policy-attributes AttributeName=ProxyProtocol,AttributeValue=true`
    2. Attach policy to load balancer
        `aws elb set-load-balancer-policies-for-backend-server --load-balancer-name <LOAD BALANCER NAME FROM AWS OR SCALR> \
         --instance-port 80 --policy-names Lab-ProxyProtocol-policy`
    3. Verify configuration
        `aws elb describe-load-balancers --load-balancer-names <LOAD BALANCER NAME FROM AWS OR SCALR>`
    4. Configure health check
        `aws elb configure-health-check --load-balancer-name <LOAD BALANCER NAME FROM AWS OR SCALR> \
         --health-check Target=TCP:80,Interval=30,UnhealthyThreshold=5,HealthyThreshold=3,Timeout=5`

### Configuring back end servers
There are a few options here.

1. Configure Nginx to read the Proxy Protocol header and then pass it on to your backend service which can be Node, etc... Google nginx proxy protocol or review https://www.built.io/blog/2014/11/websockets-on-aws-elb/
2. Configure Node using findhit-proxywrap https://github.com/findhit/proxywrap

For live teaching we chose option #2

### Reversing the changes to ELB are a bit tricky, so here are the steps
     
     1. Unset the policy from the load balancer
        `aws elb set-load-balancer-policies-for-backend-server --policy-names [] \
         --load-balancer-name <LOAD BALANCER NAME FROM AWS OR SCALR> --instance-port 80`
     2. Delete the policy
        `aws elb delete-load-balancer-policy --load-balancer-name <LOAD BALANCER NAME FROM AWS OR SCALR>\
         --policy-name <POLICY NAME>`
