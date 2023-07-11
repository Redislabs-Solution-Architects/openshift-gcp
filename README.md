# openshift-gcp 
Has the instructions to set up an OpenShift Container Platform (OCP) cluster on GCP, then create a Redis Enterprise cluster and Redis database, 
and eventually an Active-Active environment (2 mirrored Redis database clusters on 2 OpenShift clusters)

This repo started from here: https://docs.google.com/document/d/15ZlhHxgv2tiJYx_9x4eDDRiz9YCp44X0FT_r1EFtFuA/edit?usp=sharing

## Installing OpenShift on jump host
While you can install and run OpenShift on a Mac laptop, these instructions have you create a jump host in GCP for this.

This process is much like creating a "compute engine" for Redis Enterprise Software on GCP. I'll outline my steps, but
there are many variations on this theme you can take.

1. Navigate to [Google Console](https://console.google.com) and login with your Redis cloud account credentials.

2. Create a VM instance. I'm using Ubuntu on an e2-standard-4. The key is to select a network interface that you've already created that has the right firewall policies
        
    Something like this: [Firewall rules](./resources/firewall-rules.png)
3. Have the Google Cloud SDK installed on your laptop as you'll need that to at least transfer files. A call to
    `gcloud auth list`
    Should tell you if you do have it installed.
4. Now comes the fun part...installing OpenShift. You'll need to create a RedHat account, which I found to be a pain. You have to enter personal information numerous times before it lets you at the downloads page here:
    [OpenShift Downloads](https://console.redhat.com/openshift/downloads)
5. At this point I ssh to the jump host.
6. Download the OpenShift command-line interface (oc) for Linux x86_65 https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
    
    I used `wget` to download the file on my jump host.
7. Download the OpenShift Installer for Linux x86_64 https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
8. Download or copy your `Pull Secret` which you'll use later. You'll need to get that to your jump host. Either copy/paste in a `vi` session or use `scp`
9. You need to generate an SSH secret for the installer to ssh into the other servers it creates. You'll copy the contents of the id_rsa.pub it generates into the config yaml file later
   ```
   # Take the defaults. It will create two files in ~/.ssh - id_rsa and id_rsa.pub
   ssh-keygen
   ```
10. Install the oc cli by untarring the client tar file.

    ```
    tar xvf openshift-client-linux.tar.gz
    ```
     Rename the README.md if you want to refer to it later.

### Service account key
You'll need to generate a key (json) from the proper service account with all the right permissions. 
Acquiring that is beyond the scope of this document, but your manager (or their manager) should be able to get you that.
You'll need to `scp` that file to the jump host.

### Installing OpenShift
First, export your credentials environment variable set to the credentials file you generated or were given:
```
export GOOGLE_APPLICATION_CREDENTIALS=<file.json>
```
Next, extract the installer
```
tar xvf openshift-install-linux.tar.gz
```
Next, download the supporting yaml config files. We've given you a head start, but you'll have to edit them to set
names and regions specific to your environment. Search for `REPLACE_ME` in the various files found in this repo under `resources`
```
git clone https://github.com/Redislabs-Solution-Architects/openshift-gcp.git
```

Create a new folder for the cluster config and copy the cluster1 config file there
```
mkdir openshift-cluster1-conf
cp openshift-gcp/install-config.yaml_cluster1 openshift-cluster1-conf/install-config.yaml
# Update the copied file, install-config.yaml, with your names and region, pull secret and ssh public key.
```

Create the OpenShift cluster. This may take more than 30 minutes. Things will fail quickly if you don't have the right permissions.
```
./openshift-install create cluster --dir ~/openshift-cluster1-conf
```
