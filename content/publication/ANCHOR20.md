---

title: "Anchor: Diskless Cluster Provisioning Using Container Tools"
type: publication
layout: single
date: 2020
description: |
  Voss, J. (2020). Anchor: Diskless Cluster Provisioning Using Container Tools.
  Presented at SC '20: International Conference for High Performance Computing,
  Networking, Storage and Analysis. Atlanta, GA, USA.
  https://sc20.supercomputing.org/proceedings/sotp/sotp_pages/sotp107.html
bibtex: /bib/anchor20.bib
 
---

Large scale compute clusters are often managed without local disks to ease
configuration management across several hundred nodes. This diskless management
frequently relies on a collection of in-house scripts designed to build client
compute images. By leveraging container building tools matured by the tech
community we can reduce internal technical debt while allowing cluster
installations to be more flexible and resilient. Deploying container images to
compute clusters, however, remains an unsolved problem. To this end we present
Anchor, an extensible initrd module designed to boot clusters from an immutable
squashFS image with a read-write overlay. The code referenced in this paper is
available at [https://github.com/olcf/anchor](https://github.com/olcf/anchor).

* [Paper](https://sc20.supercomputing.org/proceedings/sotp/sotp_files/sotp107s2-file1.pdf)
* [Slides](https://sc20.supercomputing.org/proceedings/sotp/sotp_files/sotp107s2-file2.pdf)
