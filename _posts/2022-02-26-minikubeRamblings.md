---
title: Minikube Project Ramblings
---
## The Obstacle is the Way
I've been working with Minikube the past couple weeks and the learning curve has been steep. My goal was to get Jenkins up and running. I got it installed but while I was trying to expose the ingress for my Jenkins service I realized that Minikube may not do what I need it to.  The problem I had was in exposing Minikube to the Internet and while reading the docs I realized that just isn't what it's for. I still want to migrate some of my personal workloads to Kubernetes but my initial goal was to learn Jenkins so I'm going to choose an easier installation method and focus on that for the time being.

I wouldn't consider the experience to be a waste. My understanding of Kubernetes before starting this project was only at a very high level. I learned about the building blocks of a cluster; pods, services, ingresses, and a bit about how nodes interact with each other. When I revisit the project I'd like to run a worker node on my home network and another in AWS. For the time being, I'll probably end up running Jenkins in Docker.

## Future Plans
I've got a lot of project ideas in my head but here is a list of them roughly in order of priority:
* **Home Network Glow Up** - I have cat5 sprawled all over my house and new floors coming soon. I'm taking the opportunity to wire 2-3 bedrooms for ethernet and move all my networking equipment to a closet. I'll make sure to take pictures as I go and do a write up when I'm finished. 
* **Learn Go** - I decided I want to learn Go when I found out that Kubernetes and Terraform Providers are written in Go, Ken Thompson helped design it, and just how fast it is. I plan on plugging away at the [2021 Advent of Code](https://adventofcode.com/2021) which I started in December and put on pause to study for my AWS Developer Associate.
* **Learn Jenkins** - I really want some hands on experience with CI/CD so I plan to spin up a Jenkins server and maybe start a new project with Go to try it out on.
* **Contribute to an Open Source Project** - I found out about [Google Summer of Code](https://summerofcode.withgoogle.com/) and I think I want to take a crack at it. If I don't make the cut I'll contribute to Debian either the sponsored maintainer route, fixing bugs, or just helping to write documentation.

I'm happy to say I passed my developer cert but I'm not really sure what to go after next. I'm leaning towards the AWS SysOps Associate just to have the 3 associate certs knocked out but I'm also interested in RHCSA, CKA, and Certified Scrum Master. A mentor would be great right now. I'm not short on motivation but I think some advice would go a long way. Can't complain though, I love this stuff. Every day I learn a little bit more and I get a little bit closer to whatever's next for me.
