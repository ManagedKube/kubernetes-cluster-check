# kubernetes-cluster-check

The problem that this project tackles is that after you bring up a new cluster, update the cluster in some form (Kubernetes version, changes to secure it down, cloud keys/access, etc) or make any other infrastructure changes, how do you know that "everything" is working?  Kubernetes has a lot of features and interactions with services internally and to your cloud.  How do you perform an e2e test to make sure all of these items are working?

This project aims to help you check your Kubernetes cluster(s) to ensure that all of it's functionality is working as expected.  It will run through a check list of items to make sure each functionality is working as expected and report on what does not.  This will give you confidence that your changes didn't break anything.

For example:
- What if you changed the AWS IAM permission for your nodes to create ELBs for a `service.type: LoadBalancer`?  You won't know until the next time you need to create a load balancer.  If that breaks is in production, that is not the time you want to find out.
- Some changes to the Kubernetes API Audit Log settings could render it to send logs to the wrong directory or not outputting logs at all.  You might not notice this type of change unless you were specifically looking for it and testing for it.
- Testing a statefulset that spans multiple zones.  Testing end to end that the pods can spin up on nodes in different zones.  The cluster can provision EBS volumes and attach them successfully (https://www.youtube.com/watch?v=1xHmCrd8Qn8 time: 22:14).  This should work but there might be some mis-configuration which might make it not work.
- Test external-dns is working by adding a new dns entry and checking if that DNS entry gets created and then deleted
- Tests cert-manager is working by adding a new cert request for http01 and dns01 validation
- Test prometheus `ServiceMonitor`, adding a service to be scraped and checking if prometheus adds it in as a target and if the metrics shows up in prometheus by querying it
- Check that the cluster-autoscaler is working by provisioning out pods and making sure a new node is started and the pods gets provisioned
- Creating a nginx-ingress controller which turns up an ELB.  Checking that the end to end path works from the ELB to the nginx to the pod

This project was inspired by:
- My own personal need as a consultant.  Clients needed a way to test out their cluster after a change.
- This blog and kubecon talks is pretty much what this project wants to do but on a more generic level for Kops, EKS, GKE, and AKS clusters:  https://srcco.de/posts/how-zalando-manages-140-kubernetes-clusters.html


# Why automated Kubernetes updates and rollout?
Another reason for a Kubernetes cluster automated pipeline to deploy, test, and then roll all master and worker nodes in all enviroment is that you want this to be a routine activity.  The Kubernetes cluster is a very foundational and important piece to your software stack.  If there is a problem on this level, everything running on it will be affected by it.  Which means, you really want to get this part right.

Once you start to have multiple clusters for dev, qa, staging, prod.  Or multiple prod clusters in different region the "update" problem gets bigger.  If you are doing this manually, it takes time from your team to coordinate the updates, roll it out, roll out the updates to the node groups, and then testing it all out.  This set of activity has to be applied to all clusters.  Then if you have to wait for maintenance periods, and you have a bunch of clusters this process can take days if not week to fully roll out.  

The problem then becomes, you have someone spending mental energy to track the update changes and performing the rollout.  If someone does this every other day, this person will have to context switch back and forth and remember what was done or look at some work log to see where to pick it up from again.  The problem gets worst if this task has to switch to another person.  The other person has to then understand the process and where to pick it up from and what to do next.  This is a lot of mental engery being used.

To further compound the problem is that if this process to fully roll out an update to all of the clusters takes days or weeks (yeah weeks, if there are certain maintenance period when these activities can happen and you have to wait for these time windows) and during that time there is a critical security update that needs to go out ASAP.  For one, you can't roll out something to all the Kubernetes cluster ASAP and two, the previous rollout is in mid state.  Then to get this in, you have to do two set of activities.  Test the patch for the previous version and the new version.  Then do two types of rollout while freezing the current rollout. Yeah, you see how this gets complex?  Now you have to dedicate a person to manage this process.  That is time taken away from being able to do other things.

The Zolander blog presents a very good point and solution to this problem.  They basically mention the above faults and their solution is to automate this process and put testing in place to have confidence that "mostly" rolling this out in an automated fashion is a "safe" activity to do.  They also go with the philosophy of this update activity is always happening on the clusters and the application should be use to the cluster nodes restarting or pods restarting and moving around all the time and be able to handle this type of environment.

If your application is able to handle nodes restarting and shuffling the pods around and you had the tests in place to have confidence that the cluster updates are going well, then you can start to not have set maintenance windows for these types of updates and you can just roll these updates in whenever you want.  

You might not have the automated pipeline that they do and you might just have the tests where you can run it manually at first but by just having the tests will give you confidence that the rollout is happening correctly and that the tests are passing.  Then you start to rollout cluster changes faster.  Instead of taking days or weeks, you can do a bunch of them in parallel and maybe do them in a few hours or a day. 






![the workflow](/images/draw.io/kubernetes-automated-deployment.png)


