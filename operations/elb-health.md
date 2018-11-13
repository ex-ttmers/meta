# ELB Health

We have seen cases where performance has degraded because instances have not successfully registered with our load balancer (ELB). Even though the servers have started up, no requests are being sent to them since the load balancer doesn't know about them. Then the servers that the ELB DOES know about start getting overloaded as traffic increases.

To check if this is the case:

1. Log in to your AWS account or ask someone who has one. Currently, Andre, Jeff, and Carol have these. Talk to Andre if you find you need one.
1. Make sure you are in the US East/N. Virginia region in the upper right.
1. From the AWS Management Console, go to the EC2 section.
1. In the list of resources we're using, click on 'Load Balancers'.
1. Click on the load balancer with the largest number of instances; this one should be production.
1. In the lower panel, click on the "Instances" tab.
1. The 'Status' column should say 'InService' if all of the instances are registered correctly, and 'OutOfService' if not.

If there are a number of out of service instances, spin up more instances:

1. Log in to scalr
1. Click on 'Farms' in the top menu
1. In the "Production" row, click on the number in the "Roles" column (7)
1. In the "RailsAppServer" row, open the "Actions" menu and choose "Launch New Server".

If the new instances register with the ELB correctly, great! Everything should be happy now! If they don't, it's time to escalate to AWS.

Andre can also change the autoscaling rule to spin up more servers, but because the production farm is locked, he's the only one who can do that (which you might want to do if you want to keep the out of service instances around for investigation).
