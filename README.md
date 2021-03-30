# Kanso Carbon Model

This repository contains the source code of the Kanso Carbon Model. You'll fin below its documentation and the different scientific sources it refers to.

Hereafter is the summary of this documentation.

0. Modeling attempt on the carbon footprint of a data center
1. Emissions from running the IT room of a data center


## 0. Modeling attempt on the carbon footprint of a data center

For the sake of our estimation, we consider that the carbon footprint of a data center can be further subdivided in four different carbon footprints:
1. Emissions from running the IT room of a data center
2. Emissions from manufacturing IT equipments and air conditioners
3. Emissions from transferring data from the data center to other data centers and to the internet
4. Embodied emissions from building the data center


## 1. Emissions from running the IT room of a data center

Our model includes emissions due to IT equipments, air conditioning and electrical losses and is suddivided as emissions from:
  * A. Running compute primitives (EC2)
  * B. Running storage primitives (EBS, S3, ...)
  * C. Running IT room network devices
  * D. Air conditioning and electrical losses

### A. Running compute primitives (EC2)

This source is among the major sources of carbon emissions then we chose to have an approach based on several methods. As of today we have based our estimation model on two methods so the output of our model is a range of carbon emissions. We will keep updaing our model to enrich it with further methods and more recent data.

#### A.1 Running compute primitives (EC2) - Teads & D. Guyon

We've named this first approach after the work conducted by [Teads' engineering team](https://www.teads.com/) through [Estimating AWS EC2 Instances Power Consumption](https://medium.com/teads-engineering/estimating-aws-ec2-instances-power-consumption-c9745e347959) and David Guyon with his scientific research document [Supporting energy-awareness for cloud users. Networking and Internet Architecture](https://tel.archives-ouvertes.fr/tel-01973083/file/GUYON_David.pdf). We calculate emissions due to the run of compute primitives with the following formula:









* [1] [Estimating AWS EC2 Instances Power Consumption](https://medium.com/teads-engineering/estimating-aws-ec2-instances-power-consumption-c9745e347959), Teads, March 2021
* [2] David Guyon. [Supporting energy-awareness for cloud users. Networking and Internet Architecture](https://tel.archives-ouvertes.fr/tel-01973083/file/GUYON_David.pdf) [cs.NI]. Université Rennes 1, 2018. English. ffNNT : 2018REN1S037ff. fftel-01973083f

## Model documentation

- [AWS EC2 instances](./doc/aws-ec2-instances.md)

## Datasets

- [AWS EC2 CPU information](./data/aws-ec2-cpu-information/README.md)
