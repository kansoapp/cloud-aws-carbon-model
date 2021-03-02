# AWS EC2 carbon model

## Model overview
We didn't re-invent the wheel to come up with this model and leverage the excellent work performed by Benjamin Davy and his team at Teads. Their work is well described in [this article](https://teads-engineering-web.s3.eu-west-1.amazonaws.com/aws_instance_specifications.csv.zip), referenced later as _TEADS01_.

They came up with this formula to estimate emissions of EC2 instances:

```
EC2 running hours * Instance Ratio * Physical Instance Energy Consumption (kWh) * Region Emission Factors (CO2kg/kWh) * PUE
```

The next sections detail how the different parts are calculated.

## Instance Ratio
```
Instance type vCPU number / max instance family vCPU number
```

Teads provides a dataset for the values of AWS EC2 instances ([here](https://teads-engineering-web.s3.eu-west-1.amazonaws.com/aws\_instance\_specifications.csv.zip)).

## Region Emission Factors (CO2kg/kWh)

We're using the values provided in _TEADS01_ for now. In a future version, we will try to adjust with more precise values, as well as dynamically adjust using realtime sources like ElectricityMap.

- Virginia US eGRID: ~0.335 kgCO2/kWh in 2018 (reported as 739.35 lbCO2/MWh), using the “Virginia” state data on the service
- Ireland SEAI: 0.375 kgCO2/kWh in 2018
- Tokyo Bureau of Government: 0.470 kgCO2/kWh in 2017
- France eco2mix: 0.035 kgCO2/kWh in 2019

## PUE
We use the average value provided by AWS: `1.2` (source _TEADS01_)

## Physical Instance Energy Consumption (kWh)
That's the tricky part. Teads' article doesn't provide values for this metric yet, so we need to find a way to approximate this value.

_NB: Teads team is currently working on RAPL measurements that will enable to improve the estimation of this variable. It seems Thoughtworks and Spotify are also working on ways to improve estimations for AWS. As soon as their results are published, we will update our model to integrate those results._

To estimate the value of this metric for a given instance, we will rely on 2 approaches.

### SPECPower database

The [SPECPower database](https://www.spec.org/power_ssj2008/results/power_ssj2008.html) provides real measurements of servers assembled by different industry companies, evaluating the total energy consumption of a given setup in 2 load configurations:
- active and idle,
- active and with a 100% CPU load.

The reported consumptions are measured at the server's power input with a physical watt-meter. 

The limitations of this approach however are significative:
- the configurations built for these measures are certainly far from the configurations used by AWS in their datacenters and we have no way of knowing the impact on the measurements (the CPU may be configured differently, many hardware components will differ, etc.),
- the workload used for the SPECPower measurement protocol doesn't represent a real workload of a server in an AWS datacenter, which is unpredictable due to the mutualization of a server for many instances.

However, this approach may still provide an interesting magnitude order and server as a first version of our estimate.

In cases where the instance CPU would not be matchable with a configuration present in the SPECPower database, we will have to rely on an even worse approximation. 

### TDP

According to this [research article](https://www.spec.org/power_ssj2008/results/power_ssj2008.html) mentioned by Teads, the distribution of the power consumption of a server is approximately 55% for CPU (43%) and memory (12%).

As such, estimating the CPU's power consumption may also provide an approximation of the server's total consumption.

There is only one information available from CPU manufacturers that enable to "guesstimate" the CPU consumption: the TDP, or Thermal Dissipation Power.

> Thermal Design Power (TDP) represents the average power, in watts, the processor dissipates when operating at Base Frequency with all cores active under an Intel-defined, high-complexity workload.

Several articles indicates that, though it could be used as good proxy for energy consumption several years ago, it's not the case anymore.

  > For now, our advice is to take any published Intel high-end desktop TDP and multiply the claimed value by 1.5x – 3.3x for a more accurate estimate of peak CPU power consumption. Use 1.5x – 2.25x for regular high-end desktop CPUs, and 2.0x – 3.3x for lower power SKUs. This problem likely extends at least as far down the stack as the Core i5-10600 versus the Core i5-10600K. AMD reliably lands within or relatively near to its claimed TDP values, while Intel’s smallest measured excursion is nearly twice the size of AMD’s largest.
  
[Source: ExtremeTech article](https://www.extremetech.com/computing/319402-intels-desktop-tdps-no-longer-useful-to-predict-cpu-power-consumption)

For many reasons out of the scope of this article, the TDP will not represent the maximum power the CPU may draw anymore. Still, we could use it, corrected using the factors indicated in ExtremeTech's article above, to get a range of energy consumption for a given processor, and then obtain the total server's consumption for this instance using the following formula:

```
Physical Instance Energy Consumption (kWh) = 100/43 * vCPU Consumption
```

While the theoritical peak consumption of the CPU (and thus the server) may be deduced from the TDP as explained above, the minimum consumption will be based on the assumption that it is about 30% of the max (as mentioned in 2 sources: [Theodo article](https://blog.theodo.com/2020/09/power-api-deep-dive/) and a comment by Vincent Caron on [Etsy CloudJewels article](https://codeascraft.com/2020/04/23/cloud-jewels-estimating-kwh-in-the-cloud/)).

## Future improvements

To improve this model in the future, we will:

- If we make use of TDP values in case of missing processor types in the SPECPower database, we may measure the correlation between the TDP and SPECPower measurements on different families of processor to ensure the use of TDP is correct enough.
- Update the _Physical Instance Energy Consumption (kWh)_ metric using results from Teads and/or Thoughtworks measurements of AWS EC2 instances.
- Adjust the estimation according to the utilization.

## References

- [TEADS01](https://medium.com/teads-engineering/evaluating-the-carbon-footprint-of-a-software-platform-hosted-in-the-cloud-e716e14e060c)
- [Etsy CloudJevels](https://codeascraft.com/2020/04/23/cloud-jewels-estimating-kwh-in-the-cloud/)

## Glossary
**RAPL**
Running Average Power Limit
- It's a measure of CPU consumption implemented in Intel processors from Sandy Bridge processors. 
- It's also implemented on AMD processors Ryzen and Epic (not exhaustive) ([source](https://community.amd.com/t5/server-gurus-discussions/what-exact-parts-of-the-cpu-are-recorded-by-the-rapl-registers/td-p/73498))
- It may be accessed at software level for bare-metal servers, not for virtualized instances. [Scaphandre by Hubblo](https://github.com/hubblo-org/scaphandre/) provides tooling to retrieve this data and for datacenter operators to provide it to their VM customers.