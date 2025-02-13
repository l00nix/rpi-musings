# Intro

Inspired by “Network Chuck”’s video tutorial[ “i built a Raspberry Pi SUPER COMPUTER!! // ft.
Kubernetes (k3s cluster w/ Rancher)” on YouTube](https://www.youtube.com/watch?v=X9fSMGkjtug){:target="_blank"} and the corresponding course on his [website](
https://learn.networkchuck.com/courses/take/raspberry-pi-projects/lessons/29763478-i-built-a-
raspberry-pi-super-computer-ft-kubernetes-k3s-cluster-w-rancher/discussions/index){:target="_blank"}, as well as Jeff Geerling’s [video channel](https://www.youtube.com/@JeffGeerling){:target="_blank"} and [blog](https://www.jeffgeerling.com/blog){:target="_blank"}, I set out on four different learning experience paths:

1. The first led me down my father’s footsteps of electrical engineering, designing and building a custom carrier board for the Raspberry Pi compute module.
2. The second, was designing a cluster tray that would house my cluster in a unique way, inspired by toilet paper rolls and a Pringles tube. But more about that later.
2. The third, learning and creating kubernetes clusters on Raspberry Pis with [kubernetes](https://kubernetes.io/){:target="_blank"}/[k3s](https://k3s.io/){:target="_blank"} - a light weight version of kubernetes and managing them with [Rancher](https://www.rancher.com/){:target="_blank"}. 
3. The fourth, learning a new to me way of automating/simplifying administrative tasks, cluster installs, and management using [Ansible](https://www.ansible.com/){:target="_blank"}.

I should add that I am well down the path of my career, but I have always enjoyed tech and Linux. However, I am doing all of this as a hobby. I am sure there are many different (and possibly better) ways of doing things, but this was my way.

# 1 - Developing the LiM Cluster CM4 Carrier Board
Watching  Jeff Geerling's video [New Raspberry Pi Projects - CM4 NAS, Piunora, and Seaberry!](https://www.youtube.com/watch?v=7Li7Nh9V74I&t=300s){:target="_blank"} where he talks about [mebs_t's self designed NAS](https://github.com/mebs/CM4-NAS){:target="_blank"}, made me think I am interested in doing something like this too. At the time, the use case I was working on was [staking Navcoin headlessly with a Raspberry Pi Compute Module 4.](/2021/06/13/navcoin-staking/){:target="_blank"}

I was looking for a Raspberry Pi Compute Module 4 (CM4) carrier board that only provides power to a CM4 with WiFi and emmc storage. Many of the carrier boards out at the time provided a lot more functionality, so set out to design and build the [Less-is-More (LiM) carrier board](https://lim.loonix.ca/pages/LiM_Board.html){:target="_blank"}.

What I was looking for was a minimalistic board and I had a conversation with [Jeff Geerling](https://www.jeffgeerling.com/){:target="_blank"} when I asked him if he knew of a more minimalistic design than [this board](https://www.tindie.com/products/dronecz/minimal-carrier-board-for-compute-module-4/){:target="_blank"}? He pointed me to uptime.lab's [Upberry](https://www.instagram.com/p/CPGakesLwBo/){:target="_blank"}.

Neither of these boards are exactly what I was looking for for my use case. So I was motivated to design my own, customized board to my specs and that's how the Less-is-More board series came to be. 

## The RPi CM4 - LiM Carrier Board series

The original idea of the LiM Carrier Board series was to build a series of minimalistic Raspberry Pi (RPi) Compute Module 4 (CM4) Carrier Boards. Less-is-More (LiM) refers to the minimalistic design only providing the most rudimentary functionality to the CM4 such as 5V power via USB-C power connector and two status (power/activity) LEDs for the original LiM Carrier Board version. An additional LiM+ version of the LiM Carrier Board featured additional functionality by adding flashing capability through a jumper.

Except for the LiM CM4 Cluster Carrier Board, the LiM and LiM+ Carrier boards are meant for CM4 boards with onboard storage (the LiM CM4 Cluster Carrier Board supports both CM4 versions with and w/o eMMC (lite version), micro SD card, and 2232 and has a 2241 M.2 M-key socket for NVMe PCIe SSD) and WiFi as the LiM and LiM+ carrier boards provide no other functionality other than power and status LEDs and flashing capability (LiM+). The LiM CM4 Cluster Carrier Board has PoE Ethernet and supports WiFi and non-WiFi CM4 models.

{% include carousel.html height="75" unit="%" duration="7" number="1" %}

<p style="font:Arial;text-align:center;margin-top: -20px; color: $grey-color; font-weight: bold; font-size: 14px">

Collage 0: Various LiM Carrier Board models

</p>

Three versions of the LiM Carrier Board series have been prototyped, and more information can be found on their respective pages.

- [LiM Carrier Board](https://lim.loonix.ca/pages/LiM_Board.html){:target="_blank"}
- [LiM+ Carrier Board](https://lim.loonix.ca/pages/LiM+_Board.html){:target="_blank"}
- [LiM CM4 Cluster Carrier Board Carrier Board](https://lim.loonix.ca/pages/pages/LiM_Cluster_CB.html){:target="_blank"}

The original LiM carrier board took about two months from idea (May 22nd, 2021) to delivery (July 14th, 2021). Special thanks to [Anish Verma aka. thelasthandyman](https://www.fiverr.com/thelasthandyman){:target="_blank"} and [Muhammad S.](https://www.upwork.com/freelancers/~0152bdd89a2bb116f1){:target="_blank"} for working with me on the CAD designs of the carrier board.

I forked this board design from [Shawn Hymel](https://github.com/ShawnHymel/rpi-cm4-base-carrier){:target="_blank"}. He has a two part YouTube series where he goes through how to design a CM4 Carrier Board.

- [Part 1 - How to Make a Raspberry Pi Compute Module 4 Carrier Board in KiCad](https://www.youtube.com/watch?v=ypcPJC_umPQ){:target="_blank"}
- [Part 2 - How to Make a Raspberry Pi Compute Module 4 Carrier Board in KiCad](https://www.youtube.com/watch?v=ge6gYIENo8Q&t){:target="_blank"}

Feel free to modify this design for your own application. 

Here is the current spec and feature list of designed and planned carrier boards:

Model | Power (5V USB-C) | LEDs (Power/Activity) | Flashing Capability |
|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|
LiM Board | &#10004; | &#10004; | &#10060; |
LiM+ Board | &#10004; | &#10004; | &#10004; |

While working on the LiM+ Board, I got interested in clustering and decided to switch directions. I wanted to add a couple more features that could be of value for a cluster type carrier board. And so, the LiM CM4 Cluster Carrier Board idea was born. The LiM CM4 Cluster Board has the following features:

{::nomarkdown}<ul><li>RPi Pi Compute Module 4 (CM4) support (both versions with and w/o eMMC - Lite version)</li><li>PoE Gigabit RJ45 port (GbE)</li><li>USB-C (USB 2.0) for CM4 module flashing (power + data)</li><li>toggle switch for boot/flash mode</li><li>microSD card slot</li><li>2232 and 2241 M.2 M-key socket dedicated for NVMe PCIe SSD drives</li><li>Fan Pins</li><li>SDA, SDC, YCC, GND pins for OLED Displays</li></ul>{:/} 

This summarizes how I had the idea, designed and built a custom carrier board for the my LiM cluster board.

### The manufacturing process
<p style="font:Arial;text-align:center; color: red; font-weight: bold; font-size: 14px">

Start Sponsored Content

</p>

Now that I had the designs drawn up for the LiM cluster board, I needed them manufactured. I chose [PCBWay](https://www.googleadservices.com/pagead/aclk?sa=L&ai=CG1ORm212ZPfOKrX948APobg4hPq59GrcnczZjw2xqdvDqjAQASDigucDYP2wlIHoA6ABlcGQ0wPIAQmpAgrXcpEj9YI-qAMByAPLBKoE2wFP0E5ZYbOEVHVSHlFvHghomI0QhY0mClDk6fxSNUMDCaFTterBgSkRUrqJyQc5Y3DhH4i0SQkBaSgn28dkFzsthVpjhpI067n8zmCzPIFYuCSS_7VsxKPpA4pa8irKRcq8968gKx7MWUtrmfWjGyseDevGG37sEG6KzBtS-iTzbZQJhJjCbu7y9QxjqdmccGZQt09G_ZwkOEPkgzizZpdEw946khxUgIU4XnP8zH_LHZAUjfYhUsSQT6nls3UIIvjH3GaeZpdKp9Vbwfjh9jUBvKSlcwO9FQW2nXjABLPn0dogoAYugAfTvu8sqAeOzhuoB5PYG6gH7paxAqgH_p6xAqgHpKOxAqgH1ckbqAemvhuoB5oGqAfz0RuoB5bYG6gHqpuxAqgHg62xAqgH_56xAqgH35-xAtgHANIIGQiAYRABGB4yAooCOgeZ0ICAgIAESL39wTqxCdTiCMcjE8bogAoBmAsByAsBgAwBuAwB2BMCiBQF0BUBmBYB-BYBgBcB&ae=1&num=1&cid=CAQSOwBygQiDClAQHCteU5wwnHPRC9RgpdHwDARsDE3Da0QiBZ8LinLV9_oOLFYXnwCAG0U8-oQbfDebs329GAE&sig=AOD64_0t0Ad6fuG55d_1XRKTqS0I0D1rrA&client=ca-pub-9992192301854026&rf=1&nb=9&adurl=https://www.pcbway.com/orderonline.aspx%3Fadzy%3D19%26campaignid%3D172250251%26adgroupid%3D8780018611){:target="_blank"}.

{% include image.html
            img="/images/pcbway.png"
            title="PCBWay Ad"
	    url="https://www.googleadservices.com/pagead/aclk?sa=L&ai=CG1ORm212ZPfOKrX948APobg4hPq59GrcnczZjw2xqdvDqjAQASDigucDYP2wlIHoA6ABlcGQ0wPIAQmpAgrXcpEj9YI-qAMByAPLBKoE2wFP0E5ZYbOEVHVSHlFvHghomI0QhY0mClDk6fxSNUMDCaFTterBgSkRUrqJyQc5Y3DhH4i0SQkBaSgn28dkFzsthVpjhpI067n8zmCzPIFYuCSS_7VsxKPpA4pa8irKRcq8968gKx7MWUtrmfWjGyseDevGG37sEG6KzBtS-iTzbZQJhJjCbu7y9QxjqdmccGZQt09G_ZwkOEPkgzizZpdEw946khxUgIU4XnP8zH_LHZAUjfYhUsSQT6nls3UIIvjH3GaeZpdKp9Vbwfjh9jUBvKSlcwO9FQW2nXjABLPn0dogoAYugAfTvu8sqAeOzhuoB5PYG6gH7paxAqgH_p6xAqgHpKOxAqgH1ckbqAemvhuoB5oGqAfz0RuoB5bYG6gHqpuxAqgHg62xAqgH_56xAqgH35-xAtgHANIIGQiAYRABGB4yAooCOgeZ0ICAgIAESL39wTqxCdTiCMcjE8bogAoBmAsByAsBgAwBuAwB2BMCiBQF0BUBmBYB-BYBgBcB&ae=1&num=1&cid=CAQSOwBygQiDClAQHCteU5wwnHPRC9RgpdHwDARsDE3Da0QiBZ8LinLV9_oOLFYXnwCAG0U8-oQbfDebs329GAE&sig=AOD64_0t0Ad6fuG55d_1XRKTqS0I0D1rrA&client=ca-pub-9992192301854026&rf=1&nb=9&adurl=https://www.pcbway.com/orderonline.aspx%3Fadzy%3D19%26campaignid%3D172250251%26adgroupid%3D8780018611"
            caption="Sponsored PCBWay Ad" %}

PCBWay is a China-based manufacturer specializing in PCB (Printed Circuit Board) production and assembly. They serve both hobbyists and professionals, offering a variety of services such as PCB prototyping, small-batch production, PCB assembly, and other related services.

The company is well-regarded for their competitive pricing, range of options (such as different board materials and finishes), and a user-friendly online ordering system which allows customers to get instant quotes and track their orders.

The company supports a variety of file formats for board design, and they also offer design and layout services. Additionally, PCBWay runs a community platform where PCB designers can share their projects and participate in contests.

There are certainly other services out there but from my personal experience, I can only recommend their PCB manufacturing services. Pricing is reasonable and their customer services is great. I got clarifying emails from their technicians to ensure the PCBs are being manufactured correctly. Even though with the different timezones, they are responsive leading to minimal delays in the manufacturing process. Shipping is also fast, however I had to pay duty and import fees (which you have to with all other manufactures as well, so this is not a PCBWay issue).

<p style="font:Arial;text-align:center; color: red; font-weight: bold; font-size: 14px">

End Sponsored Content

</p>
