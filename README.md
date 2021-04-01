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

#### A.1 Running compute primitives (EC2) - The _Teads & D. Guyon's_ approach

We've named this first approach after the work conducted by [Teads' engineering team](https://www.teads.com/) through [Estimating AWS EC2 Instances Power Consumption](https://medium.com/teads-engineering/estimating-aws-ec2-instances-power-consumption-c9745e347959) and David Guyon with his scientific research document [Supporting energy-awareness for cloud users. Networking and Internet Architecture](https://tel.archives-ouvertes.fr/tel-01973083/file/GUYON_David.pdf). We calculate emissions due to the run of compute primitives with the following formula:

![A 1](https://user-images.githubusercontent.com/8396084/113326153-98aef380-9319-11eb-82da-ee930ee94c6a.jpg)


We have determined the *Share of the CPU in the energy consumption of the Physical Machine* out of David Guyon's work. The distribution of the energy consumption page 33 highlights that 43% of the energy consumed by a physical machine is linked to the CPU. David Guyon cites *D. Kliazovich, P. Bouvry, and S. U. Khan, “GreenCloud: A Packet-level Simulator of Energyaware Cloud Computing Data Centers,” The Journal of Supercomputing, vol. 62, no. 3, pp. 1263–1283, 2012* which was published in 2012. This scientific base is quite old yet:
- As David Guyon noted, this is the most recent paper he found on this subject in the literature;
- We can certainely assume that the consumption distribution inside a physical machine hasn't evolved much over the past years.

Then the rest of the calculation stands as following:

![A 1 2](https://user-images.githubusercontent.com/8396084/113325958-5be2fc80-9319-11eb-9e18-5975af350914.jpg)

With:
- *i* beeing the type of instance considered (c5, m5, r5, ...)
- *g* beeing the geographies where the observed cloud infrastructure have at least one instance running 
- *v(i,g)* beeing the total volume of hours run by *i* instances in the *g* geography
- *p(i)* beeing the power required by the instance's CPU at a standard 40%[3] server utilization
- *c(g)* beeing the carbon intensity of the geography *g* of the data center considered


#### A.2 Running compute primitives (EC2) - The _NRDC's_ approach

We have named this second approach after the report published by the Natural Resources Defense Council (NRDC) in 2014 called Data Center Efficiency Assessment. A part of their study is focused on the level of utilization of IT equipment and we could find , the impact of and potential for efficiency opportunities in multitenant data centers, and the degree to which the evolution of the industry’s technology and delivery model is aligning incentives to further drive energy efficiency







* [1] [Estimating AWS EC2 Instances Power Consumption](https://medium.com/teads-engineering/estimating-aws-ec2-instances-power-consumption-c9745e347959), Teads, March 2021
* [2] David Guyon. [Supporting energy-awareness for cloud users. Networking and Internet Architecture](https://tel.archives-ouvertes.fr/tel-01973083/file/GUYON_David.pdf) [cs.NI]. Université Rennes 1, 2018. English. ffNNT : 2018REN1S037ff. fftel-01973083f
* [3] Data Center Efficiency Assessment, 2014, Appendix 2, NRDC.org

## Model documentation

- [AWS EC2 instances](./doc/aws-ec2-instances.md)

## Datasets

- [AWS EC2 CPU information](./data/aws-ec2-cpu-information/README.md)
