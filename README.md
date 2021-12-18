# pi-cluster
Raspberry Pi Cluster Perpetual Repo

An effort to move what services I could onto my 4 Node k3s Raspberry Pi 4 Bramble while also getting more familiar with Kubernetes over other tooling I'd used before.

This repo will always be tailored for running everything through kubectl, though may add other dependencies as I go along like using kustomize and jsonnet to further familiarity with these tools.

## Pre-Requisites / Assumptions
- Using Raspberry PiOS 64-bit
- Running k3s
- Master node has enough storage to share to the nodes for NFS
- Have an NFS server setup (either with really fast connected LAN or on the master node... Mines on the master node)
- Provisoned the `nfs-subdir-external-provisioner` (terrible name btw) and its name is `nfs-local`

## Cloudflare DDNS
I know this could be run as a cronjob but I'm familiar with this docker container from running it on Unraid.

TODO: perhaps write a bespoke implementation with busybox and api calls?

Ensure you have a secrets file (see sample), I have them stored just outside the repository (sibling folder) and draw them in as to not accidently commit my credentials to this repo.
```shell
kubectl apply -f cloudflare-ddns/ -f ../secrets/cloudflare.yaml
```

## Unbound
I like hosting my own DNS at edge, and unbound has never let me down. It's happy being Ephemeral so its happy just being a `Deployment`.

Only tricky stuff is that upstream ingestors (lancache and adguard home) both do not like resolving the kubernetes service address i.e. `unbound.unbound.svc.cluster.local` (though if forced adguard home can). As a result using a static clusterIP as it has no issues in the config files and UI's of these.

```shell
kubectl apply -f unbound/
```

## Lancache
With multiple PCs at home pulling Windows updates and using Steam/GamePass/Origin/etc, this is a no-brainer for me.. And that I have a 30TB+ Unraid box with enough free space to cache all the games I own... I like it...

This repo does not cover setting up the LanCache Monolith server, which you should run on another machine... Unless you're doing this on a PiNAS that is...

See here for how to set this up: https://lancache.net/docs/

There is an Unraid native template from the Community Apps plugin to speed it up as well if you use this for your NAS

```shell
kubectl apply -f lancache-dns/
```

## Adguard Home
I hate ads... So much that I pay for YouTube Premium (and before that YouTubeRed) and pay for any other native "ad free" stuff where I can.
For the rest... there's AdGuardHome...

Why not PiHole... I mean, you're running this on Raspberry Pi's? 

Well, I ran it for years and I never quite got it right and it drove my partner crazy (I guess me too) where on Google it would display the sponsored ad links, but you click on them and it would not resolve the dns... 

The internet is broken enough without this brand of BS... I tried AdGuard Home on a whim and it performs perfectly vs PiHole so I've kept it.

There are some caveats with this one... 
- You have the nfs subdir provisioner setup
- It's running as a `StatefulSet` as it has some state it likes to keep hanging around
- Adguard home does not like using the svc.cluster.local dns to resolve the upstream dns servers, so they need to be static `clusterIP`s
- I have some LAN rewrites in there, suggest removing/adding your own

```shell
kubectl apply -f adguardhome/
```

## Kubernetes Dashboard
I was using Rancher, and while I like it I dont like losing 40%+ of my limited resources on my Pi Cluster so enter Kubernetes Dashboard...
This is a work in progress... I cannot get the ingress working on it for seemingly no reason...
