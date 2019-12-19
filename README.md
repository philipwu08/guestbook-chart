# Guestbook sample application helm chart installation through a channel

This example describes how to use MCM Channel and Subscription to install a helm chart from ICP local helm chart repository into a managed cluster in a multi-cluster environment. A channel will be created to connect to the ICP MCM hub cluster's local helm chart chart repository https://`<ICP-hostname>`:8443/helm-repo/charts, the guestbook sample application helm chart will be loaded into the helm repository and a subscription will be created to subscribe to the channel for installing the guestbook application helm chart into a managed cluster. `default` namespace will be used.


## Usage

1. Clone this Git repository and package the guest book application helm chart. `helm package gbapp` helm package gbchn
2. Command-line login to the MCM hub cluster and load the helm chart. `cloudctl catalog load-chart --archive gbapp-0.1.0.tgz`
3. Open the `channel.yaml` file and replace the `<ICP_hostname>` with MCM hub cluster's hostname or IP address.
4. Run `kubectl apply -f channel.yaml` to create a channel that point to the ICP MCM hub cluster's local helm chart repository. It also associates the channel with `skip-cert-verify` config map that indicates to the channel to skip SSL certificate validation because the local helm repository has a self-signed certificate.
5. In the managed cluster, the Helm CRD controller that installs subscribed helm charts also requires the `skip-cert-verify` config map when the source helm repository has a self-signed certificate. Command-line login to the managed cluster and run `kubectl apply -f skip-cert-configmap.yaml`. 
6. Run `kubectl apply -f image-policy.yaml` to create a image policy for importing guesbook docker images from public docker image repositories. 

**Note:** Propagation of this config map and image policy to all managed clusters could be automated by creating a Namespace type channel, creating this config map and image policy in that channel and using subscriptions to subcribe to the config map and image policy from the channel.

7. Log back into the MCM hub cluster and create the subscription to get the sample guesbook application helm chart intalled into the managed cluster. `kubectl apply -f subscription.yaml`. 
8. After a several minutes, you can log into the managed cluster's `default` namespace to observe with `kubectl get subscription` command that `guestbook-subscription` subscription is created with status `Subscribed`.  Also `gbapp-guestbook` helm release is installed with redis-master, redis-slave and guesbook frontend deployments. 
