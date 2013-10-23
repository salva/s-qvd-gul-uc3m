# QVD

<img width="40%" src="images/qvd-logo.png"/>

Salvador Fandiño (salvador@qindel.es)

QindelGroup

---

# QVD

- Producto de "Virtual Desktop Infrastructure" (VDI)

- 100% Open Source

- 100% Linux

---

# Cliente

- Linux
- Windows
- OS X
- Android
- iOS (en desarrollo)
- Plugin Firefox/Chrome (en desarrollo)

---

<img src="images/windows-client.png"/>

---

# Escritorio virtual

- Linux
    + Gnome
    + KDE
    + **[LXDE](http://lxde.org)**
    + **[XFCE](http://www.xfce.org)**
    + [IceWM](http://www.icewm.org/)
    + Otros

---

<img src="images/gnome-desktop.png"/>

---

# Servidor/Granja

- Linux
    + Ubuntu
    + SuSE
    + RedHat

---

# Administración

- Herramienta Web

- Herramienta de linea de comandos

---

<img src="images/wat-dashboard.png"/>

---

# Destaca por...

- Eficiencia de la solución (CPU, RAM, I/O y ancho de banda).
- Escalabilidad (10.000 usuarios por granja, 400 VMs por nodo).
- Gestión de imagenes de disco inspirado en los sistemas de control de versiones.
- Integrable y adaptable (Open Source, arquitectura).
- Tambien funciona como servidor de aplicaciones.

---

# Arquitectura

<img src="images/qvd-network.png"/>

---

# Por dentro

- Linux
- Virtualizadores KVM y LXC
- HTTP/SSL para la comunicacion cliente-servidor
- NX para compression del protocolo X11
- iptables y ebtables para confinamiento de las VMs
- PostgreSQL
- Desarrollado en Perl, C, Javascript

---

# ¿Preguntas?

---



---

# Exprimiendo Linux

...o como llegar a correr 400 máquinas virtuales en un host.

---

# Cuellos de botella

- **RAM**
- **I/O**
- CPU
- Red
- **Kernel**

---

<img src="images/bottlenecks-ram-vs-cpu.jpg"/>

---

# Interdependencias

- Si aumenta el uso de memoria por los procesos &rarr; menos memoria
  para buffer de disco &rarr; aumenta el I/O
- Si aumenta el acceso al disco por los procesos &rarr; aumenta el uso
  de memoria como buffers &rarr; aumenta el uso de RAM
- Si aumenta el uso de swap &rarr; aumenta la carga de I/O

---

# Máquinas virtuales

- KVM, Xen, VMWare, etc.
- La emulación introduce sobrecarga de CPU e I/O
- Cada máquina virtual tiene su propio kernel.
- Las VMs no colaboran sino que compiten por los recursos.
- Cada VM tiene sus buffers de I/O
- **Las VMs usan el 100% de la memoria asignada**.

---

# VMs - Soporte hardware

Cada nueva generación de procesadores introduce mejoras en el soporte
para virtualización:
[AMD-V, VT-x, AMD-Vi, VT-d, EPT, VMCS](http://en.wikipedia.org/wiki/X86_virtualization)

---

# VMs - [Virtio](http://www.linux-kvm.org/page/Virtio)

- Reemplaza la emulación de hardware por hiper-llamadas desde el
  kernel de la máquina virtual al sistema anfitrión
- Reduce el uso de CPU
- Soportado por KVM, Xen, VMware, etc.

---

# VMs - [KSM](http://www.linux-kvm.org/page/KSM)

- Consolida páginas de memoria con igual contenido.
- Funciona con KVM
- El consumo de CPU no es despreciable &dash; O(N).
- Muy efectivo con los buffers de I/O.
- No detecta páginas que no estan siendo utilizadas.
- Poco efectivo con la memoria de procesos que usan JITs,
  garbage-collectors o muchas estructuras dinamicas (JVM, GTK+).

---

# VMs - Balloon driver

- Pincha un agujero en la RAM de la máquina virtual y
  se la devuelve al sistema anfitrión.
- Soportado en KVM, Xen, VMWare, ...
- No introduce carga de CPU.
- **Se adapta a la "presión" de memoria dentro de la máquina
    virtual**.
- Las VMs usan su swap &rarr; I/O&uarr; **¡peligro!**

---

<img width="70%" src="images/balloon.jpg"/>

---

# VMs - I/O

- Uso total de disco
- Número de operaciones por segundo
- x 2 &rarr; local, remoto

## Sin perder de vista...

- Provisión de VMs
- VMs volatiles

---

# VMs - Overlays, [qcow2](http://en.wikipedia.org/wiki/Qcow)

- Sobre una unica imagen base, se crean varias imagenes derivadas.
- Es una capa adicional con coste en CPU e I/O.
- Los overlays solo almacenan las diferencias con la base.
- Podemos almacenar el overlay en local si no necesitamos
  persistencia.

---

# VMs - Optimización del SO invitado

- Se puede optimizar los sistemas operativos que corren en las VMs
- Eliminar I/O innecesario (caches navegador, logs).
- Utilizar aplicaciones ligeras (IceWM).
- Separar partes persistentes (`$HOME`) de partes desechables (`/`).
- El `cron`...

---

# VMs - Conclusiones

- Mejor soporte hardware, virtio &rarr; CPU&darr;
- KSM &rarr; RAM&darr; CPU&uarr;
- Balloon driver &rarr; RAM&darr; I/O&uarr;
- Overlays &rarr; Uso de disco total&darr;&darr;
- Redistribuir carga de I/O entre local y remoto.
- Optimización del SO virtualizado.

¡No hay soluciones mágicas!

---

# [LXC](http://en.wikipedia.org/wiki/LXC) - Linux Containers

> chroot on asteroids

---

# Linux, el kernel...

Permite particionar los recursos que gestiona:

- Procesos (`/proc`)
- Arbol de sistemas de ficheros
- Red (interfaces, rutas, reglas firewall, etc.)
- ...

---

# LXC

- Crea contenedores utilizando las capacidades de particionamiento del
  kernel.
- Aun en desarrollo. Tiene bugs, normalmente poco criticos y oscuros.
- Similar a Solaris zones, FreeBSD jails, Linux-VServer, OpenVZ.

---

# LXC

- Funcionalmente un contenedor es identico a una máquina virtual.
- Con la particularidad de que no ejecuta su propio kernel.
- Control de uso de CPU, RAM e I/O
  ([cgroups](http://en.wikipedia.org/wiki/Cgroups)).
- Puede compartir recursos con el anfitrión &rarr; ¡flexibilidad!
- Al menos tan seguro como la virtualización completa.

---

# LXC en 20 segundos

    # como root...
    lxc-create -n test -t sshd
    lxc-ls
    mkdir /var/lib/lxc/test/rootfs/root/.ssh
    cp ~salva/.ssh/id_dsa.pub /var/lib/lxc/test/rootfs/root/.ssh/authorized_keys
    lxc-start -n test

    # en otra terminal...
    ssh 10.0.3.23 -l root    

---

# LXC - Ventajas

- No hay sobrecarga de RAM, CPU o I/O.
- Con LXC solo hay un kernel que tiene visibilidad de todo y se
  encarga de repartir los recursos entre los contenedores/procesos
  eficazmente.
- Se comparten los buffers de I/O.
- Los contenedores **solo usan la memoria que necesitan**.

---

# LXC - Inconvenientes

- Se usa el kernel del sistema anfitrión.
    + En teoria malo para las actualizaciones.
    + En la práctica no es un problema.
- No tenemos overlays de disco.

---

# LXC - `s/overlays/?/`

---

# LXC - LVM?

- Con LVM podemos emular los overlays.
- Gestión muy complicada.
- Cada capa es vista de manera independiente por el kernel &rarr; los
  datos acaban duplicados en buffers de I/O.

---

# LXC - [Btrfs](http://en.wikipedia.org/wiki/Btrfs)?

- Solo funciona en local.
- Crear nuevas máquinas es muy rápido.
- No hay datos duplicados en disco (a nivel de bloque).
- Se comparten buffers de I/O pero no `mmap`s.
- En desarrollo, hace falta un kernel muy reciente.
- SuSE lo soporta en producción.

---

# LXC - `mount --bind`?

- Una rama del sistema de ficheros se puede "remontar" en otro lugar
  (enlace simbolico a lo bestia).
- `mount --bind` rutas `ro` (`/**/{sbin,bin,lib}`) + `untar` rutas `rw` (`/{var,etc,...}`) &rarr; imagen OS!
- Se comparten buffers de I/O y **`mmap`s**.
- Requiere una imagen muy modificada.

---

# LXC - union file systems?

- [UnionFS](http://www.fsl.cs.sunysb.edu/project-unionfs.html)
- UnionFsFuse
- [AuFS](http://aufs.sourceforge.net/)
- [Union Mounts](http://valerieaurora.org/union/)
- [OverlayFS](https://kernel.googlesource.com/pub/scm/linux/kernel/git/mszeredi/vfs/+/overlayfs.current/Documentation/filesystems/overlayfs.txt)

¡el culebrón!

---

# Union File Systems

- Muchos casos de uso distintos.
- Compromiso entre eficiencia, funcionalidad y
  compatibilidad (semantica POSIX).
- Nunca se consigue contentar a todo el mundo.
- Algunos casos extremos son difíciles de abordar.

---

# UnionFS

- Primer intento serio &dash; problemas de diseño importantes.
- Se reescribio, v2 &dash; aun con problemas.
- Lleva casi dos años sin actualizaciones.

---

# UnionFsFuse

- Funciona con [FUSE](http://fuse.sourceforge.net/) &rarr; no hay que
  parchear el kernel.
- Rendimiento pésimo, inviable en la práctica.

---

# AuFS

- Fork de UnionFS, reescrito completamente en v2.
- "Feature rich".
- Han intentado integrarlo en Linux varias veces sin exito.
- Parche gigantesco del kernel. Codigo dudoso.
- **Integrado en Ubuntu**

---

# Union Mounts

- Implementado sobre VFS. Estrictamente, no es un FS.
- Simplicidad pesa más que funcionalidad.
- Parecia a punto de ser integrado en 2009, pero...

---

# OverlayFS

- Simplicidad sobre funcionalidad.
- A punto de ser integrado en 3.10... ¡Aun hay esperanzas!
- **Integrado en Ubuntu**
- Con LXC, se comparten buffers de I/O y **`mmap`s**.

---

# En resumen...

- Imagenes persistentes &rarr; OverlayFS* y AuFS en Ubuntu.
- Imagenes volatiles &rarr; Btrfs en SuSE, OverlayFS en Ubuntu.
- Para todo lo demás... no hay opciónes.

---

# VDI en Linux - conclusiones

- LXC ofrece mejores prestaciones que KVM.
- Pero el "ecosistema" Linux/LXC aun no está maduro.

---

# ¿Preguntas?

---

# ¡Gracias!

---

# Referencias

- Esta presentación: http://github.com/salva/s-qvd-gul-uc3m
- [QVD](http://theqvd.com)
- Union File Systems:
     + [When you don't need Union Mounts](http://www.linux-kongress.org/2009/slides/state_union_mount_jan_blunck.pdf)
     + [Unioning file systems: Architecture, features, and design choices](http://lwn.net/Articles/324291/)
     + [Union file systems: Implementations, part I](http://lwn.net/Articles/325369/)
     + [Unioning file systems: Implementations, part 2](http://lwn.net/Articles/327738/)
     + [A brief history of union mounts](http://lwn.net/Articles/396020/)
     + [Another union filesystem approach](http://lwn.net/Articles/403012/)
     + [Debating overlayfs](https://lwn.net/Articles/447650/)
     + [Overlayfs for 3.10](http://lwn.net/Articles/542707/)






