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
