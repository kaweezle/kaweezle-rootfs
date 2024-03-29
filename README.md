<!-- markdownlint-disable MD033 MD041 -->
<div id="top"></div>
<!-- markdownlint-enable MD033 MD041-->

<!-- PROJECT SHIELDS -->

[![Deprecated][deprecated-url]][iknite] [![Go Version][go-version]][go-version]
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url] [![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![Apache 2.0 License][license-shield]][license-url]
[![Experimental][stability]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]
[![Workflow][workflow-shield]][workflow-url]

<!-- PROJECT LOGO -->
<!-- markdownlint-disable MD033 -->

<br />
<div align="center">

  <a href="https://github.com/kaweezle/kaweezle-rootfs">
    <img src="images/logo.svg" alt="Logo" width="80" height="80">
  </a>
  <h3 align="center">Kaweezle Root Filesystem</h3>

  <p align="center">
    Run Vanilla Kubernetes on Windows with WSL 2 and Alpine Linux
    <br />
    <a href="https://github.com/kaweezle/kaweezle-rootfs"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/kaweezle/kaweezle-rootfs/issues">Report Bug</a>
    ·
    <a href="https://github.com/kaweezle/kaweezle-rootfs/issues">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#deprecation-notice">Deprecation notice</a></li>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
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
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>
<!-- markdownlint-enable MD033 -->

<!-- DEPRECATION NOTICE -->

## Deprecation notice

This project is being **DEPRECATED** starting from **January 2023**. Its main
feature, which is building the root filesystem supporting [Kaweezle], is being
transferred to [Iknite]. This project is kept for reference.

<!-- ABOUT THE PROJECT -->

## About The Project

Kaweezle allows running a Kubernetes cluster on Windows using Windows Subsystem
for Linux 2 (WSL 2).

This project is the sister project of [Kaweezle]. It contains the root
filesystem of the WSL distribution used for running Kubernetes.

The distribution is created from the Alpine
[mini root FS](https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/x86_64/) by
adding the appropriate packages from the Edge repository. The container images
of the base pods (coredns, api-server, ...) are also downloaded for faster setup
times.

The distribution contains a go based executable, `iknite` that is run by its
Windows counterpart (`kaweezle`) to start or restart the appropriate
dependencies and the Kubernetes cluster.

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

### Built With

This project uses the following components:

- [go](https://go.dev/)
- [cobra](https://github.com/spf13/cobra)
- [logrus](github.com/sirupsen/logrus)
- [client-go](https://github.com/kubernetes/client-go)

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Getting Started

Please refer to the
[kaweezle readme](https://github.com/kaweezle/kaweezle/README.md) for
installation instructions.

The following sections give instructions on how to use the root filesystem
without the `kaweezle` command.

### Prerequisites

To run kaweezle, you'll need to have
[WSL installed](https://docs.microsoft.com/en-us/windows/wsl/install).

The simplest way to install it is to run the following command:

```powershell
> wsl --install
```

After reboot, update the kernel and set the default version to version 2:

```powershell
> sudo wsl --update
> wsl --set-default-version 2
```

For the other tools, you can use [Scoop](https://scoop.sh/) or
[Chocolatey](https://chocolatey.org/).

To use the kubernetes cluster, you will need to have kubectl installed:

```powershell
> scoop install kubectl
```

Other tools might be of interest, like `k9s`, `kubectx`, `kubens` or `stern`.
All are available through scoop. You can install all of them at once with the
following command:

```powershell
> scoop install k9s kubectx kubens stern
```

### Download and installation

The root filesystem can be downloaded from the
[Releases](https://github.com/kaweezle/kaweezle-rootfs/releases) page.

You can create a WSL distribution with the following set of commands:

```powershell
> cd $env:LocalAppData
> mkdir kwsl
> cd kwsl
> (New-Object Net.WebClient).DownloadFile("https://github.com/kaweezle/kaweezle-rootfs/releases/download/latest/rootfs.tar.gz", "$PWD\rootfs.tar.gz")
> wsl --import kwsl . rootfs.tar.gz
```

You will have a WSL distribution called `kwsl` which file system will be located
in the current director:

```powershell
❯ wsl -l -v
  NAME                    STATE           VERSION
* Alpine                  Stopped         2
  kwsl                    Stopped         2
```

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->

## Usage

To start the kubernetes cluster, issue the following command:

```powershell
❯ wsl -d kwsl iknite -v info start -w 120
```

The distribution is now running:

```powershell
❯ wsl -l -v
  NAME                    STATE           VERSION
* Alpine                  Stopped         2
  kwsl                    Running         2
```

Now the kubernetes cluster can be accessed:

<!-- cSpell: disable -->

```powershell
❯ $env:KUBECONFIG="\\wsl$\kwsl\root\.kube\config"
❯ kubectl get nodes
NAME              STATUS   ROLES    AGE   VERSION
laptop-vkhdd5jr   Ready    <none>   61s   v1.23.1
❯ kubectl get pod --all-namespaces
NAMESPACE            NAME                                      READY   STATUS    RESTARTS   AGE
kube-system          coredns-64897985d-bhhzq                   1/1     Running   0          68s
kube-system          coredns-64897985d-mvpbb                   1/1     Running   0          68s
kube-system          etcd-laptop-vkhdd5jr                      1/1     Running   0          84s
kube-system          kube-apiserver-laptop-vkhdd5jr            1/1     Running   0          84s
kube-system          kube-controller-manager-laptop-vkhdd5jr   1/1     Running   0          84s
kube-system          kube-flannel-ds-hkz9p                     1/1     Running   0          68s
kube-system          kube-proxy-xx5xp                          1/1     Running   0          68s
kube-system          kube-scheduler-laptop-vkhdd5jr            1/1     Running   0          78s
kube-system          metrics-server-d9c898cdf-7qbbr            1/1     Running   0          68s
local-path-storage   local-path-provisioner-566b877b9c-qnpzx   1/1     Running   0          68s
metallb-system       controller-7cf77c64fb-h72jx               1/1     Running   0          68s
metallb-system       speaker-2h66l                             1/1     Running   0          68s
```

<!-- cSpell: enable -->

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- ROADMAP -->

## Roadmap

See the [open issues](https://github.com/kaweezle/kaweezle-rootfs/issues) for a
full list of proposed features (and known issues).

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- CONTRIBUTING -->

## Contributing

Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and
create a pull request. You can also simply open an issue with the tag
"enhancement". Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- LICENSE -->

## License

Distributed under the Apache License. See `LICENSE` for more information.

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- CONTACT -->

## Contact

Kaweezle - [@kaweezle](https://twitter.com/kaweezle)

Project Link:
[https://github.com/kaweezle/kaweezle-rootfs](https://github.com/kaweezle/kaweezle-rootfs)

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->

## Acknowledgements

This project started from the amazing work made by
[yuk7](https://github.com/yuk7) with [wsldl](https://github.com/yuk7/wsldl) and
[AlpineWSL](https://github.com/yuk7/AlpineWSL).

It also uses the great work made by the Alpine Linux community on the edge
repository.

You may be interested by existing alternatives from which we have taken some
ideas:

- [Rancher Desktop](https://rancherdesktop.io/)
- [Minikube](https://github.com/kubernetes/minikube)
- [Kind](https://kind.sigs.k8s.io/)

By using
[kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
and Alpine, kaweezle is closer to the clusters you may use on public clouds.

This readme has has been created from the
[Best-README-Template](https://github.com/othneildrew/Best-README-Template)
project.

<!-- markdownlint-disable-line --><p align="right">(<a href="#top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

<!-- prettier-ignore-start -->

[contributors-shield]: https://img.shields.io/github/contributors/kaweezle/kaweezle-rootfs.svg?style=for-the-badge
[contributors-url]: https://github.com/kaweezle/kaweezle-rootfs/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/kaweezle/kaweezle-rootfs.svg?style=for-the-badge
[forks-url]: https://github.com/kaweezle/kaweezle-rootfs/network/members
[stars-shield]: https://img.shields.io/github/stars/kaweezle/kaweezle-rootfs.svg?style=for-the-badge
[stars-url]: https://github.com/kaweezle/kaweezle-rootfs/stargazers
[issues-shield]: https://img.shields.io/github/issues/kaweezle/kaweezle-rootfs.svg?style=for-the-badge
[issues-url]: https://github.com/kaweezle/kaweezle-rootfs/issues
[license-shield]: https://img.shields.io/badge/license-apache_2.0-green?style=for-the-badge&logo=none
[license-url]: https://github.com/kaweezle/kaweezle-rootfs/blob/master/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/kaweezle
[go-version]: https://img.shields.io/badge/Go-1.17+-00ADD8?style=for-the-badge&logo=go
[stability]: https://img.shields.io/badge/stability-experimental-orange?style=for-the-badge
[workflow-shield]: https://github.com/kaweezle/kaweezle-rootfs/actions/workflows/build.yaml/badge.svg
[workflow-url]: https://github.com/kaweezle/kaweezle-rootfs/actions/workflows/build.yaml
[deprecated-url]: https://img.shields.io/badge/DEPRECATED-kaweezle%2Fikinte-inactive?style=for-the-badge&logo=github
[Kaweezle]: https://github.com/kaweezle/kaweezle
[Iknite]: https://github.com/kaweezle/iknite

<!-- prettier-ignore-end -->
