<a id="readme-top"></a>

[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![Apache License][license-shield]][license-url]



<br />

<h3 align="center">PivotLab</h3>

  <p align="center">
A Ludus range for practicing pivoting, lateral movement, and initial access techniques. Originally created by <a href="https://github.com/CleverNamesTaken/PivotLab">CleverNamesTaken</a>, this fork extends the lab with a dual-homed network topology, a Windows jump box with Responder/LLMNR poisoning for initial access, and additional exploitation scenarios.
</p>

</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#network-diagram">Network Diagram</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

Broadly speaking, this lab includes the use of three different types of tools:

- live off the land tools
- non-native binaries
- webshells

The lab consists of three dual-homed jump boxes and two target machines across two network segments. The jump boxes include a Tomcat server for JSP webshells, an Apache server for PHP webshells, and a Windows Server running IIS (for ASPX webshells) with a Responder/LLMNR poisoning bait. The Kali attack box can only reach the jump boxes on VLAN 20 — to reach the targets on VLAN 22, you must pivot through the jump boxes. Linux machines can be administered via the SSH key on the Kali box. The Windows jump box has a weak service account (`svc_backup`/`butterfly`) that broadcasts LLMNR traffic for hash capture with Responder.

### Network Diagram

The jump boxes are **dual-homed** -- they have a NIC on VLAN 20 (attacker network) and a second NIC on VLAN 22 (target network). The Kali attack box can only reach VLAN 20. To reach the targets on VLAN 22, you must pivot through one or more jump boxes.

```
                         VLAN 20                                      VLAN 22
                   (Attacker Network)                            (Target Network)
                    10.X.20.0/24                                  10.X.22.0/24

  ┌──────────────┐                    ┌──────────────────┐
  │     Kali     │                    │   lamp (Apache)  │
  │  Attack Box  │   10.X.20.220      │                  │  10.X.22.220
  │ 10.X.20.201  ├───────────────────►│ eth0        eth1 ├──────────┐
  │              │                    │  PHP webshells   │          │
  └──────┬───────┘                    └──────────────────┘          │
         │                                                          │    ┌──────────────────┐
         │                            ┌──────────────────┐          ├───►│   Linux Target   │
         │           10.X.20.221      │  windows-jump    │          │    │   10.X.22.50     │
         ├───────────────────────────►│  (IIS/WinRM)     │  10.X.22.221  │                  │
         │                            │ nic0        nic1 ├──────────┤    │ CVE-2024-47176   │
         │                            │  ASPX webshells  │          │    │ CVE-2025-32433   │
         │                            │  LLMNR/Responder │          │    └──────────────────┘
         │                            └──────────────────┘          │
         │                                                          │
         │                            ┌──────────────────┐          │    ┌──────────────────┐
         │           10.X.20.222      │  tom (Tomcat)    │          │    │  Windows Target  │
         └───────────────────────────►│                  │  10.X.22.222  │   10.X.22.60     │
                                      │ eth0        eth1 ├──────────┘    │                  │
                                      │  JSP webshells   │               │ CVE-2022-3229    │
                                      └──────────────────┘               └──────────────────┘

  Firewall Rules:
    - Kali (VLAN 20) CANNOT reach VLAN 22 directly (inter-VLAN default: REJECT)
    - Jump boxes (.220-.222) CAN reach VLAN 22 (allowed by firewall rule)
    - Targets on VLAN 22 CAN respond to jump boxes
    - All VMs have WireGuard and internet access
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>


## Getting Started

This guide assumes the user already has Promox and Ludus installed.  If Ludus is not yet installed, start [here](https://docs.ludus.cloud/docs/quick-start/install-ludus).

### Installation


1. Clone the repo
   ```sh
   git clone https://github.com/BLTSEC/PivotLab.git
   ```
2. Add the necessary roles
   ```sh
   cd PivotLab
   ludus ansible role add -d roles/attack_box/
   ludus ansible role add -d roles/fvarovillodres.lamp/
   ludus ansible role add -d roles/lamp/
   ludus ansible role add -d roles/linux_target/
   ludus ansible role add -d roles/tom/
   ludus ansible role add -d roles/tomcat/
   ludus ansible role add -d roles/windows_jump/
   ludus ansible role add -d roles/windows_target/
   ludus ansible role add -d roles/ludus_vulhub/
   ludus ansible role add -d roles/dual_home/
   ```
   > **Note:** The `dual_home` role is bundled locally in this repo. If you're building your own range and want dual-homing, you can install it standalone with `ludus ansible role add BLTSEC.ludus_dual_home`. See [ludus_dual_home](https://github.com/BLTSEC/ludus_dual_home) for details.

3. Import the range config file
   ```sh
   ludus range config set -f range-config.yml
   ```
4. Deploy the range
   ```sh
   ludus range deploy
   ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Usage

Feel free to test any number of pivoting tools you'd like.  The `Conops.md` file contains a walkthrough on basic usage of the following tools, which are pre-installed on the Kali attack box at 10.<RANGE_NUMBER>.20.201:

- ssh
- iptables
- socat
- [ligolo-ng](https://github.com/nicocha30/ligolo-ng)
- [gost](https://github.com/ginuerzh/gost)
- [Chisel](https://github.com/jpillora/chisel)
- [SSF](https://securesocketfunneling.github.io/ssf/#home)
- [sshuttle](https://github.com/sshuttle/sshuttle)
- [suo5](https://github.com/zema1/suo5)
- [Neo-reGeorg](https://github.com/L-codes/Neo-reGeorg)
- [weevely3](https://github.com/epinna/weevely3.git)

ssh to 10.<RANGE_NUMBER>.20.201 with the credentials `kali:kali`, and check out the `~/tools` directory for pre-installed tools.

If you are like me and prefer your own attack box, then just run `prepareTools.sh` to install the tools on a different platform.

See `Conops.md` for how these tools can be deployed.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Top contributors:

<a href="https://github.com/BLTSEC/PivotLab/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=BLTSEC/PivotLab" alt="contrib.rocks image" />
</a>



<!-- LICENSE -->
## License

Distributed under the Apache License 2.0. See `LICENSE` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Project Link: [https://github.com/BLTSEC/PivotLab](https://github.com/BLTSEC/PivotLab)

Original Project: [https://github.com/CleverNamesTaken/PivotLab](https://github.com/CleverNamesTaken/PivotLab)

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* [CleverNamesTaken](https://github.com/CleverNamesTaken/PivotLab) for creating the original PivotLab
* [Erik (Bad Sector Labs)](https://gitlab.com/badsectorlabs) for all the amazing work on Ludus
* opsdisk and the incredible [Cyber Plumber's Handbook](https://github.com/opsdisk/the_cyber_plumbers_handbook)
* fvarovillodres for his development of the [ansible-role for installing a LAMP stack](https://github.com/fvarovillodres/ansible-role-lamp)


<p align="right">(<a href="#readme-top">back to top</a>)</p>


[contributors-shield]: https://img.shields.io/github/contributors/BLTSEC/PivotLab.svg?style=for-the-badge
[contributors-url]: https://github.com/BLTSEC/PivotLab/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/BLTSEC/PivotLab.svg?style=for-the-badge
[forks-url]: https://github.com/BLTSEC/PivotLab/network/members
[stars-shield]: https://img.shields.io/github/stars/BLTSEC/PivotLab.svg?style=for-the-badge
[stars-url]: https://github.com/BLTSEC/PivotLab/stargazers
[issues-shield]: https://img.shields.io/github/issues/BLTSEC/PivotLab.svg?style=for-the-badge
[issues-url]: https://github.com/BLTSEC/PivotLab/issues
[license-shield]: https://img.shields.io/github/license/BLTSEC/PivotLab.svg?style=for-the-badge
[license-url]: https://github.com/BLTSEC/PivotLab/blob/master/LICENSE
