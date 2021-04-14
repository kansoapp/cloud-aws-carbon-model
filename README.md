# 🌳 Kanso Carbon Model

This repository contains the source code of the Kanso Carbon Model whose goal is to estimate the carbon footprint associated with a given company's cloud infrastructure. In this readme file we tried to spell out as clearly as possible the methodology we built to approach the estimation.

# 📖 Summary
* 📌 **Important disclaimer**
* 🧮 **Modeling attempt on the carbon footprint related to the consumption of data center's services *(4 parts)***
* 🙏 **Procedure to suggest changes to enrich the methodology & record of past changes**
* 🧬 **Scientific sources this methodology is based on**

------------------------------------------------------------------------

# 📌 Important disclaimer

Tens of hours were spent to gather the scientific sources we neeeded and to build this methodology. Yet hundreds of hours would have possibly been needed to bring a more accurate method! One of our core principles at [Kanso](http://www.kansoapp.com) is that *understanding precedes action* so we favoured to build a model which is super easy to deploy within companies to maximize the spreading of the word and the *understanding*.

We have dedicated a part of this *readme* to the *Procedure to suggest changes to enrich the methodology* to ensure the update process is clear and easy for people willing to contribute. Additionally records of changes are beeing kept in this same part.



# 🧮 Modeling attempt on the carbon footprint related to the consumption of data center's services

For the sake of our estimation, we consider that the carbon footprint of a data center can be further subdivided in four different carbon footprints:
1. Emissions from consuming public cloud's services
2. Emissions from manufacturing IT equipments and air conditioners
3. Emissions from transferring data from the data center to other data centers and to the internet
4. Embodied emissions from building the data center


## 1. Emissions from consuming public cloud's services

Our model is limited to emissions due to IT equipments, air conditioning and electrical losses and is suddivided as emissions from:
  * A. Running compute primitives (EC2)
  * B. Running storage primitives (EBS, S3, ...)
  * C. Running IT room network devices
  * D. Air conditioning and electrical losses

with `1 = A + B + C + D`

### A. Running compute primitives (EC2)

This source is among the major sources of carbon emissions then we chose to have an approach based on several methods. As of today:
* we have based our estimation model on two methods so the output is a range of carbon emissions.
* we assumed the underlying data centers are *hyper-scale*. We know there is a gap between this assumption and the reality but the gap is closing as more and more data centers in the world are *hyper-scale* (see [4]).
* both our methods use the *average server utilization in a hyper-scale data center*. The NRDC 2014 report (see [3]) shows this figure is equal to 40% while the report produced by the Berkeley National Laboratory (see [6]) relates 50%.


We will keep on updating our model to enrich it with further methods and more recent data.


#### A.1 Running compute primitives (EC2) - The _Teads & D. Guyon's_ approach

We've named this first approach after the work conducted by [Teads' engineering team](https://www.teads.com/) in their article *Estimating AWS EC2 Instances Power Consumption* (see [1]) and David Guyon in his scientific research document *Supporting energy-awareness for cloud users. Networking and Internet Architecture* (see [2]). We calculate emissions due to the run of compute primitives with the following formula:

![A 1](https://user-images.githubusercontent.com/8396084/113326153-98aef380-9319-11eb-82da-ee930ee94c6a.jpg)


We have determined the *Share of the CPU in the energy consumption of the Physical Machine* out of David Guyon's work. The distribution of the energy consumption page 33 highlights that 43% of the energy consumed by a physical machine is linked to the CPU. David Guyon cites *D. Kliazovich, P. Bouvry, and S. U. Khan, “GreenCloud: A Packet-level Simulator of Energyaware Cloud Computing Data Centers,” The Journal of Supercomputing, vol. 62, no. 3, pp. 1263–1283, 2012* which was published in 2012. This scientific base is quite old yet:
- As David Guyon noted, this is the most recent paper he found on this subject in the literature;
- We can certainely assume that the consumption distribution inside a physical machine hasn't evolved much over the past years.

Then the rest of the calculation stands as following:

![image](https://user-images.githubusercontent.com/8396084/114671888-de4dc200-9d04-11eb-9607-4b023395860a.png)



With:
- *i* beeing the type of instance considered (c5, m5, r5, ...)
- *g* beeing the geographies where the observed cloud infrastructure has at least one instance running 
- *v(i,g)* beeing the total volume of hours run by *i* instances in the *g* geography
- *p(i)* beeing the power required by the instance's CPU at a 40-50% server utilization rate
- *c(g)* beeing the carbon intensity of the electricity in the geography *g* where the considered data center is located


#### A.2 Running compute primitives (EC2) - The _NRDC's_ approach

We have named this second approach after the report published by the Natural Resources Defense Council (NRDC) in 2014 called Data Center Efficiency Assessment. A part of their study is focused on the level of utilization of IT equipment in data centers and we could find the following information: *for hyper-scale data centers, the server power at average utilization level stands at 101 watts*.

Then, we calculate A.2 as below:

![image](https://user-images.githubusercontent.com/8396084/114672734-c0cd2800-9d05-11eb-8f6c-bec0ea010a67.png)

With:
- *i* beeing the type of instance considered (c5, m5, r5, ...)
- *g* beeing the geographies where the observed cloud infrastructure have at least one instance running 
- *v(i,g)* beeing the total volume of hours run by *i* instances in the *g* geography
- *r(i)* beeing the instance ratio. The instance ratio is the ratio between the vCPU allocated when the *i* instance is beeing consumed divided by the total vCPU available on the physical machine. For example, a r5 instance has an *instance ratio* of 1/96.
- *c(g)* beeing the carbon intensity of the electricity in the geography *g* where the considered data center is located



### B. Running storage primitives (EBS, S3, ...)

To estimate the energy consumed by storage primitives, we used the research work conducted by the Etsy team (see [5]) which estimated how much energy it takes to store a terabyte of data on HDD (hard disk drive) or SSD (solid-state drive) disks in a cloud computing environment:
* 0.89 Wh/TBh for HDD storage
* 1.52 Wh/TBh for SSD storage

We are interested in the order of magnitude so we estimate the emissions due to running the storage primitives as follows:
![image](https://user-images.githubusercontent.com/8396084/113697759-efc71680-96d3-11eb-8830-a957553b8c09.png)
With:
- *g* beeing the geographies where the observed cloud infrastructure have at least one storage primitive
- *d(g)* beeing the volume of data stored multiplied by the duration of storage in the *g* geography: ![image](https://user-images.githubusercontent.com/8396084/113698155-711ea900-96d4-11eb-8763-9660b2fefcdf.png)
- *c(g)* beeing the carbon intensity of the electricity in the geography *g* where the considered data center is located


### C. Running IT room network devices

We used the work from David Guyon (see [2]) to estimate the energy consumed by the network devices located in the IT room. More specifically, we based our model on the following distribution of the energy consumed at the IT room level:
* 70% of the energy are consumed by the physical machines
* 30% are consumed by network devices.

We approached the energy consumed by the physical machines by estimating A and B so we can determine the emissions implied by running the IT room network devices with the following formula:

![image](https://user-images.githubusercontent.com/8396084/113705649-fc506c80-96dd-11eb-8cc1-b109e68b1887.png)

*N.B: Discussions with experts pointed out that the energy split has evolved towards 85%/15% or even 90%/10%. Yet as we didn't find any written source backed by a scientific approach, we chose to keep the 70%/30% ratio.*

### D. Air conditioning and electrical losses

Finally, thanks to the Power Usage Effectiveness (PUE) of the considered data centers, we can determine the CO2eq emitted because of the air conditioning and the electrical losses with the following formula:
![image](https://user-images.githubusercontent.com/8396084/113705973-649f4e00-96de-11eb-8fae-20c6bd78fc49.png)


## 2. Emissions from manufacturing IT equipments and air conditioners

To our knowledge, the best approach to estimate embodied emissions from manufacturing equipments is to use the ratio:
![image](https://user-images.githubusercontent.com/8396084/113707196-d2984500-96df-11eb-844c-e46d8b0c8c6c.png)

This ratio that we call *x* is sometimes documented and comparable between different equipments. We deep dived into two research papers (see [7] and [8]) that highlight that "80% of energy and carbon came from the operational phase of a computer, which can be assumed broadly equivalent to a server, with the remaining 20% from pre-use and decommissioning".

Then we chose to calculate *emissions from manufacturing IT equipments and air conditioners* with *x*=0.25 (doing so we assume that the *x* ratio is also valid for air conditioners).

## 3. Emissions from transferring data from the data center to other data centers and to the internet

We used data from an article published by George Kamiya on the IEA's website (see [9]) pointing out that we can use the [0.025 kWh/GB; 0.23 kWh/GB] range for our Data Transmission Energy Intensity.

Hence we obtain the following formula to estimate the emissions that occur when transferring data between data centers and between a data center and the internet:
![image](https://user-images.githubusercontent.com/8396084/113712906-0165e980-96e7-11eb-83f8-317b242ccb11.png)


With:
- *g* beeing the geographies that send data
- *g'* beeing the geographies that receive data
- *d(g,g')* beeing the total volume of data that has been transferred between the geography *g* and the geography *g'*
- *d(g)* beeing the total volume of data that has been transferred between the geography *g* and the Internet
- *D* is the Data Transmission Energy Intensity. In our model we use the range assuming that data transfers between two data centers consume less energy than transfers between a data center and the Internet.
- *c(g)* beeing the carbon intensity of the electricity in the geography *g*
- *c(g')* beeing the carbon intensity of the electricity in the geography *g'*


## 4. Embodied emissions from building the data center

Approaching the emissions embodied from building the data center is complex as data centers construction processes are very different from a technology to another. Hence we chose to approach it using the *x* ratio we already used in the part 2. To do so we used the work conducted conducted by Beth Whitehead and Deborah Andrews in March 2015 (see [8]) stating that:
> operational figures for a standard 50-year building life cycle yielded values of between 70 and 80% of the overall impact.

We assumed that the embodied emissions from building the data center counted for 20-30% of the total life cycle assessment so we worked with the following formula:

![image](https://user-images.githubusercontent.com/8396084/113836134-e77ee200-978c-11eb-9be0-cb32573eae4a.png)




# 🙏 Procedure to suggest changes and to enrich the methodology

As stated before, we'll be glad to enrich this model every time a more accurate data point or a more recent approach is brought to our attention. If you want to contribute or just say hello, please drop us a note by [email](mailto:carbon@kansoapp.com) or by directly suggesting your changes through Github.


# 🧬 Scientific sources this methodology is based on

Below is the list of the different sources that are currently used by the Kanso Carbon Model:

* [1] [Estimating AWS EC2 Instances Power Consumption](https://medium.com/teads-engineering/estimating-aws-ec2-instances-power-consumption-c9745e347959), Teads, March 2021
* [2] [Supporting energy-awareness for cloud users. Networking and Internet Architecture](https://tel.archives-ouvertes.fr/tel-01973083/file/GUYON_David.pdf), David Guyon from Université Rennes 1, 2018
* [3] [Data Center Efficiency Assessment](https://www.nrdc.org/sites/default/files/data-center-efficiency-assessment-IP.pdf), NRDC, 2014
* [4] [Data Centres and Data Transmission Networks](https://www.iea.org/reports/data-centres-and-data-transmission-networks), IEA, June 2020
* [5] [Cloud Jewels: Estimating kWh in the Cloud](https://codeascraft.com/2020/04/23/cloud-jewels-estimating-kwh-in-the-cloud/), Etsy, April 2020
* [6] [United States Data Center Energy Usage Report](https://eta.lbl.gov/publications/united-states-data-center-energy), Berkeley National Laboratory, June 2016
* [7] [New perspectives on internet electricity use in 2030](https://www.researchgate.net/publication/342643762_New_perspectives_on_internet_electricity_use_in_2030), Anders S.G. Andrae, Huawei Technologies, June 2020
* [8] [The life cycle assessment of a UK data centre](https://www.researchgate.net/publication/271710820_The_life_cycle_assessment_of_a_UK_data_centre), page 398, Beth Whitehead and Deborah Andrews, March 2015
* [9] [The carbon footprint of streaming video: fact-checking the headlines](https://www.iea.org/commentaries/the-carbon-footprint-of-streaming-video-fact-checking-the-headlines), George Kamiya, IEA, December 2020
