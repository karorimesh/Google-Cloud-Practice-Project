# VPC Networking
## Explore the default network
	gcloud compute networks describe default
1. Delete the Firewall rules
	gcloud compute firewall-rules delete
2. Delete the default network
	gcloud compute networks delete default
3. Try to create a VM instance
	gcloud compute instances create intance-1

## Create an auto mode network
1. Create an auto mode VPC network with firewall rules
	gcloud compute networks create mynetwork --project=qwiklabs-gcp-04-f5eddec1fa08 --subnet-mode=auto --bgp-routing-mode=regional

gcloud compute firewall-rules create mynetwork-allow-icmp --project=qwiklabs-gcp-04-f5eddec1fa08 --network=projects/qwiklabs-gcp-04-f5eddec1fa08/global/networks/mynetwork --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp

gcloud compute firewall-rules create mynetwork-allow-internal --project=qwiklabs-gcp-04-f5eddec1fa08 --network=projects/qwiklabs-gcp-04-f5eddec1fa08/global/networks/mynetwork --description=Allows\ connections\ from\ any\ source\ in\ the\ network\ IP\ range\ to\ any\ instance\ on\ the\ network\ using\ all\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all

gcloud compute firewall-rules create mynetwork-allow-rdp --project=qwiklabs-gcp-04-f5eddec1fa08 --network=projects/qwiklabs-gcp-04-f5eddec1fa08/global/networks/mynetwork --description=Allows\ RDP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 3389. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389

gcloud compute firewall-rules create mynetwork-allow-ssh --project=qwiklabs-gcp-04-f5eddec1fa08 --network=projects/qwiklabs-gcp-04-f5eddec1fa08/global/networks/mynetwork --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22

2. Create a VM instance in us-central1
	gcloud beta compute --project=qwiklabs-gcp-04-f5eddec1fa08 instances create mynet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --network=mynetwork  --image=debian-10-buster-v20200910 --image-project=debian-cloud  --boot-disk-type=pd-standard --boot-disk-device-name=mynet-us-vm

3. Create a VM instance in europe-west1
	gcloud beta compute --project=qwiklabs-gcp-04-f5eddec1fa08 instances create mynet-eu-vm --zone=europe-west1-c --machine-type=n1-standard-1 --network=mynetwork  --image=debian-10-buster-v20200910 --image-project=debian-cloud  --boot-disk-type=pd-standard --boot-disk-device-name=mynet-eu-vm

4. Verify connectivity for the VM instances
	gcloud compute ssh mynet-us-vm --zone us-central1-c --tunnel-through-iap
	ping -c 3 <Enter mynet-eu-vm's internal IP here>
	ping -c 3 mynet-eu-vm
	ping -c 3 <Enter mynet-eu-vm's external IP here>
	exit


5. Convert the network to a custom mode network
	gcloud compute networks update mynetwork \
    --switch-to-custom-subnet-mode

## Create custom mode networks
1. Create the managementnet network
	gcloud compute networks create managementnet --project=qwiklabs-gcp-04-f5eddec1fa08 --subnet-mode=custom --bgp-routing-mode=regional

	gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-04-f5eddec1fa08 --range=10.130.0.0/20 --network=managementnet --region=us-central1

2. Create the privatenet network
	gcloud compute networks create privatenet --subnet-mode=custom
	gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
	gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
	gcloud compute networks list
	gcloud compute networks subnets list --sort-by=NETWORK


3. Create the firewall rules for managementnet
	gcloud compute --project=qwiklabs-gcp-04-f5eddec1fa08 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0

4. Create the firewall rules for privatenet
	gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
	gcloud compute firewall-rules list --sort-by=NETWORK

5. Create the managementnet-us-vm instance
	gcloud beta compute --project=qwiklabs-gcp-04-f5eddec1fa08 instances create managementnet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=managementsubnet-us --network-tier=PREMIUM  --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=managementnet-us-vm 

6. Create the privatenet-us-vm instance
	gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us --image-family=debian-10 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=privatenet-us-vm
	gcloud compute instances list --sort-by=ZONE
	
## Explore the connectivity across networks
	gcloud compute instance describe mynet-us-vm
	gcloud compute instance describe mynet-eu-vm
	gcloud compute instance describe privatenet-us-vm
	gcloud compute instance describe managementnet-us-vm
1. Ping the external IP addresses
	gcloud compute ssh mynet-us-vm --zone us-central1-c --tunnel-through-iap
	ping -c 3 <Enter mynet-eu-vm's external IP here>
	ping -c 3 <Enter managementnet-us-vm's external IP here>
	ping -c 3 <Enter privatenet-us-vm's external IP here>

2. Ping the internal IP addresses
	ping -c 3 <Enter mynet-eu-vm's internal IP here>
	ping -c 3 <Enter managementnet-us-vm's internal IP here>
	ping -c 3 <Enter privatenet-us-vm's internal IP here>
