# Intent Based Networking with Napalm-Yang 
Imagine being able to build your host_vars, your YAML inputs, directly from the network device itself. What if you could take the native vendor CLI configuration and suck it into an application which auto-generated industry standard data models which also validated the data types being inputted (such as is the input an integer, or a string, or an IP Address).  

## Solution Overview and Benefits 
Napalm-yang leverages open source tooling and solutions which are trusted and used by network automation innovators like Google to automate at scale and precision. Napalm-yang has bi-directional functionality, auto-generating device host_vars, in an industry standard format, from the device configuration itself. Conversely, napalm-yang enables network automation engineers to update the values for the host_vars, and then auto-generate the intended configuration from the host_vars, without even needing to touch a Jinja2 template. 

Another major benefit napalm-yang brings to the table is the capability to be the bridge between brownfield legacy network devices which do not support next-gen automation APIs and more recent infrastructure which is NETCONF/Yang enabled. Since napalm-yang abstracts the device inputs into a common data model, which is compatible with OpenConfig, napalm-yang can be the glue which deploys to both brownfield CLI only devices and newer API enabled devices. Also as you refresh your infrastructure and replace legacy CLI only devices with newer API enabled devices, napalm-yang speaks the common language which both your existing infrastructure and your cutting-edge devices understand. No data model rework is needed as you upgrade your devices as there is perfect continuity between all OpenConfig supported vendor devices and the napalm-yang OpenConfig generated data model. 



## Case Study: Auto-generate YAML from Device Configuration

There are many more features and capabilities in Napalm-yang, but this post is focusing on alleviating the burden of building and maintaining a network device's YAML file. One of napalm-yang's capabilities is to load in native configuration from your brownfield network devices and auto-generate an OpenConfig compliant YAML file abstracting the vendor configuration into a vendor-neutral data model. The benefit of using napalm-yang for this function is that if the data is modeled in this fashion:

- Napalm-yang can manipulate the data using powerful python API bindings and then re-render the native configuration using the napalm-yang configuration engine. 
- Now that the configuration is represented using an OpenConfig industry standard data model, if you replace that brownfield device with a NETCONF/Yang enabled device, you can continue using the same data model no matter what vendor you choose. You now have a foot in the door for programmability, as well as a head start for maintaining that programmable workflow when you upgrade your legacy CLI only devices to API enabled NETCONF/Yang devices. 
- You are benefitting from the collective knowledge of network automation engineers around the world from the world's leading network automation companies. The OpenConfig and IETF data models are being used in production by the world's most innovative companies with great success. 

So what would this look like in a simple example? Let's assume you have the following snippet of EOS interface configuration (napalm-yang supports translating configuration from major vendors such as Cisco, Juniper, Arista):

```
interface et1
    ip address 192.168.1.1/24
    description Uplink1
    mtu 9000
    exit
interface et2
    ip address 192.168.2.1/24
    description Uplink2
    mtu 9000
    exit
```

Now napalm-yang will use an open source and extensible parsing engine to convert the above EOS vendor specific configuration into a set of vendor neutral OpenConfig compliant variables:
```yaml
---
interfaces:
  interface:
    et1:
      config:
        description: Uplink1
        mtu: 9000
      name: et1
      routed-vlan:
        ipv4:
          addresses:
            address:
              192.168.1.1:
                config:
                  ip: 192.168.1.1
                  prefix-length: 24
                ip: 192.168.1.1
    et2:
      config:
        description: Uplink2
        mtu: 9000
      name: et2
      routed-vlan:
        ipv4:
          addresses:
            address:
              192.168.2.1:
                config:
                  ip: 192.168.2.1
                  prefix-length: 24
                ip: 192.168.2.1
```

This was done through the following Python code (assuming the configuration was in a file called `eos.config`):

```
with open("eos.config", "r") as f:
    config = f.read()

running_config = napalm_yang.base.Root()
running_config.add_model(napalm_yang.models.openconfig_interfaces)
running_config.parse_config(native=[config], profile=["eos"])

print(running_config.get(filter=True))
```
The code opens the file with the configuration, starts up a napalm-yang instance, loading in the industry standard OpenConfig interface data model, and then parses the EOS configuration against the OpenConfig data model. The result is the JSON equivalent of the YAML we saw previously. 

## Conclusion

The perceptive automation engineer may wonder after the previous example, where is the Jinja template? What can we do with this YAML file without a corresponding template? The answer will be expounded upon in the next post. Basically napalm-yang enables us to not only read in configuration into a vendor neutral data model automatically, but also re-render the configuration into the original vendor CLI, or any other major vendor CLI. We can update our data model, change a few things, maybe update an interface description and then rebuild our configuration to send back to the device. Finally, since we are working within the framework of Yang models, we also have the added benefit of input validation and type checking, which is not something YAML on its own provides. 