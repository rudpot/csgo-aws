# Configure CS:GO server in AWS

## Using cloudformation

* Make sure you have an AWS account and a VPC with at least on public subnet as well an an ssh keypair; also look up your own IP to enter for the `SshAccessFromThisCidrOnly` from which the server can be accessed via ssh
* Make sure you have a steam account
* Make sure the steam account has a phone number attached
* [Create a game server license for App ID 730](https://steamcommunity.com/dev/managegameservers)
* clone this repo
* create a parameter file like this called `csgo-server-params.json` - fill in the values and save ... 
  ```json
  [
    {
        "ParameterKey": "SshAccessFromThisCidrOnly",
        "ParameterValue": "x.x.x.x/32"
    },
    {
        "ParameterKey": "CSGOVpcId",
        "ParameterValue": "vpc-12345567890"
    },
    {
        "ParameterKey": "CSGOSubnetId",
        "ParameterValue": "subnet-12345567890"
    },
    {
        "ParameterKey": "SshKeyName",
        "ParameterValue": "my-ssh-key"
    },
    {
        "ParameterKey": "CSGOServerToken",
        "ParameterValue": "STEAMSERVERTOKEN"
    },
    {
        "ParameterKey": "CSGOServerPassword",
        "ParameterValue": "p@ssw0rd"
    },
    {
        "ParameterKey": "CSGOServerName",
        "ParameterValue": "p@ssw0rd"
    }
  ]
  ```
* launch the stack via the CLI (you can also do this via the console but then you have to enter the parameters manually)
  ```bash
  aws cloudformation create-stack \
    --stack-name csgotest \
    --template-body file://csgo-server.yaml \
    --parameters file://csgo-server-params.json \
    --capabilities CAPABILITY_IAM
  ```
* once the cloudformation stack finishes building you can ssh to the machine and check on installation progress in `less /var/log/cloud-init-output.log`. Note that the maps download takes a _long_ time.
* after the download finishes a CS:GO server should be started automatically. The server should also restart on reboot. 
* to change the startup configuration edit `/home/ubuntu/server-start.sh`. In particular you might want to add `+sv_tags` to make it easier to find your server
* to see the server log output check `/var/log/csgo.log`.

## Manually

Follow this: https://medium.com/@alihusseinat/how-to-setup-a-counter-strike-go-dedicated-server-on-aws-5ca80e2acf4f

Changes:

* if you are security conscious, build a new VPC to isolate the instance
* pick m5.large (t3 is too slow)
* 50GB of disk space (server download is 28GB)
* pick newest ubuntu 20.04 LTS
* give it an elastic IP
* `sudo apt install libsdl2-2.0-0:i386` as per https://github.com/ValveSoftware/steam-for-linux/issues/7036 before steamcmd
* if shit goes wrong clear the steam cache with `rm .steam/appcache/packageinfo.vdf .steam/appcache/appinfo.vdf` as per https://www.reddit.com/r/playrust/comments/c3q3zh/error_timed_out_waiting_for_appinfo_update/
* fix startup commandline
  * as per https://forums.alliedmods.net/showthread.php?t=300773 add a password to startup command 
  * as per https://forums.alliedmods.net/showthread.php?t=300182 remove -nobots
  * as per https://gamebanana.com/gamefiles/4432 change mapgroup to random_classic
  `@reboot /home/ubuntu/csgosv/srcds_run -game csgo -console -usercon +game_type 0 +game_mode 0 +mapgroup random_classic +map de_dust2 +sv_setsteamaccount <your_login_token> +sv_password <password> -net_port_try 1`
* docs
  * https://developer.valvesoftware.com/wiki/Counter-Strike:_Global_Offensive_Dedicated_Servers
  * https://developer.valvesoftware.com/wiki/Command_Line_Options


## Notes

* [Ubuntu SSM image info](https://discourse.ubuntu.com/t/finding-ubuntu-images-with-the-aws-ssm-parameter-store/15507)
* [Working with public SSM parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-finding-public-parameters.html)

```
sudo -- sh -c 'dpkg --add-architecture i386; add-apt-repository multiverse; apt-get update; apt-get -y dist-upgrade'
sudo apt install libsdl2-2.0-0:i386
# This requires manually accepting steam license ... 
sudo apt-get install -y steamcmd
```

