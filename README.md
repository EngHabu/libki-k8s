# libki-k8s

## Installing Libki on Raspberry Pi

### Image the Memory card
Follow the official raspberry pi docs to image RaspbianOS (32bit only as of the time of this writing) into your memory card.

### [Optional] Setup headless Raspberry Pi
If desired, before ejecting the memory card, follow [the steps here](https://desertbot.io/blog/headless-raspberry-pi-4-ssh-wifi-setup) to setup a headless raspberry pi. Summarized below (on OSX):

### Enable SSH
```bash
touch /Volumes/boot/ssh
```

### Auto-join Wifi
1. touch /Volumes/boot/wpa_supplicant.conf
2. vim /Volumes/boot/wpa_supplicant.conf
3. Paste:
    ```
    country=US
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="<NETWORK-NAME>"
        psk="<NETWORK-PASSWORD>"
    }
    ```

**Congratulations**
You are now ready to eject the memory card, insert it into the raspberry pi and boot it up!

### [Optional] Cert-auth for SSH
Follow the [guide here](https://pimylifeup.com/raspberry-pi-ssh-keys/) to setup ssh with key. Steps are summarized below:
1. On your client machine (that will connect to raspberry pi), run:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
   And copy the output
2. SSH into raspberry pi
   ```bash
   ssh pi@raspberrypi.local
   ```
   default password is ``raspberry``
3. Run:
   ```bash
   install -d -m 700 ~/.ssh
   ```
4. Run:
   ```bash
   nano ~/.ssh/authorized_keys
   ```
5. Paste the contents you copied from step 1

## Setup K3s on the raspberry pi:
Follow the [guide here](https://medium.com/digital-software-architecture/raspberry-pi-install-docker-and-kubernetes-b347bff37ce) to install k3s on your raspberry pi. Steps are summarized below:
1. Run:
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker pi
   su - pi
   ```
2. Run:
   ``sudo nano /boot/cmdline.txt`` and type ``cgroup_enable=memory cgroup_memory=1`` at the end of the line.
3. Reboot the raspberry pi ``shutdown -r``
4. Run:
   ```bash
   sudo curl -sfL https://get.k3s.io | sh -s - --docker
   ```

**Congratulations** 
You have successfully installed k8s on your raspberry pi

## Installing Libki on k8s
**NOTE**

If running on raspberry pi k3s, prefix all kubectl commands with ``sudo``

1. Run:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/EngHabu/libki-k8s/main/kustomize/k8s_generated.yaml

   kubectl wait --for=condition=available --all deployments --timeout=10m || ( echo >&2 "Timed out while waiting for the deployments to become ready"; exit 1 )

   kubectl rollout status --all
   ```
2. Create admin user:
   a. Run:
      ```bash
      kubectl get pods -n libki
      ```
      And note the libki server's pod name
   b. Run:
      ```bash
      kubectl exec -n libki <pod name> -- perl script/administration/create_user.pl -u admin -p password -s
      ```
3. Port forward the libki service to validate everything is fine:
   ```bash
   kubectl port-forward -n libki service/libki 10443:443
   ```

   Or if on RPI's k3s:
   ```bash
   sudo k3s kubectl port-forward -n libki service/libki 10443:443 --address 0.0.0.0
   ```
4. Visit [localhost](http://localhost:10443/administration) or [raspberry pi](http://raspberrypi:10443/administration) and login with ``admin`` and ``password``. 
   
   **NOTE**
   
   Make sure to change your password as soon as you login.

**Congratulations**

You have a running deployment of libki.

## Follow-ups

1. Set up ingress 
2. Increase the number of replicas for the libki server
3. Setup a cluster of Raspberry PIs to ensure the k8s cluster high-availability.
