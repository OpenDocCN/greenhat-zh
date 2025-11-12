# 第十一章. 案例研究：智能平台管理接口驱动程序

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

本章探讨了 `ipmi(4)` 的部分内容，即智能平台管理接口 (IPMI) 驱动程序。IPMI 规范定义了用于监控和管理系统硬件的标准。

### 注意

对于我们的目的来说，这个 IPMI 的描述就足够了，因为本章的目的是展示 PCI 驱动程序如 `ipmi(4)` 如何使用 PMIO 和 MMIO。

`ipmi(4)` 的代码库由 10 个源文件和 1 个头文件组成。在本章中，我们将遍历其中一个文件，*ipmi_pci.c*，其中包含与 PCI 总线相关的代码。

# 代码分析

示例 11-1 提供了 *ipmi_pci.c* 的简洁、源代码级别的概述。

示例 11-1. ipmi_pci.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/bus.h>
  #include <sys/condvar.h>
  #include <sys/eventhandler.h>
  #include <sys/selinfo.h>

  #include <machine/bus.h>
  #include <sys/rman.h>
  #include <machine/resource.h>

  #include <dev/pci/pcireg.h>
  #include <dev/pci/pcivar.h>

  #include <dev/ipmi/ipmivars.h>

  static struct ipmi_ident {
          u_int16_t       vendor;
          u_int16_t       device;
          char            *description;
  } ipmi_identifiers[] = {
          { 0x1028, 0x000d, "Dell PE2650 SMIC interface" },
          { 0, 0, 0 }
  };

  const char *
  ipmi_pci_match(uint16_t vendor, uint16_t device)
  {
  ...
  }

  static int
  ipmi_pci_probe(device_t dev)
  {
  ...
  }

  static int
  ipmi_pci_attach(device_t dev)
  {
  ...
  }

  static device_method_t ipmi_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_probe,         ipmi_pci_probe),
          DEVMETHOD(device_attach,        ipmi_pci_attach),
          DEVMETHOD(device_detach,        ipmi_detach),
          { 0, 0 }
  };

  static driver_t ipmi_pci_driver = {
          "ipmi",
          ipmi_methods,
          sizeof(struct ipmi_softc)
  };

 DRIVER_MODULE(ipmi_pci, pci, ipmi_pci_driver, ipmi_devclass, 0, 0);

  static int
  ipmi2_pci_probe(device_t dev)
  {
  ...
  }

  static int
  ipmi2_pci_attach(device_t dev)
  {
  ...
  }

  static device_method_t ipmi2_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_probe,         ipmi2_pci_probe),
          DEVMETHOD(device_attach,        ipmi2_pci_attach),
          DEVMETHOD(device_detach,        ipmi_detach),
          { 0, 0 }
  };

  static driver_t ipmi2_pci_driver = {
          "ipmi",
          ipmi2_methods,
          sizeof(struct ipmi_softc)
  };

 DRIVER_MODULE(ipmi2_pci, pci, ipmi2_pci_driver, ipmi_devclass, 0, 0);
```

在我描述 示例 11-1 中的函数之前，请注意它包含两个 `DRIVER_MODULE` 调用。换句话说，示例 11-1 声明了两个 Newbus 驱动程序；每个都设计用来处理一组不同的设备（正如你很快就会看到的）。

现在，让我们讨论 示例 11-1 中找到的函数。

## ipmi_pci_probe 函数

`ipmi_pci_probe` 函数是 示例 11-1 中找到的第一个 Newbus 驱动程序的 `device_probe` 实现。以下是它的函数定义：

```
static int
ipmi_pci_probe(device_t dev)
{
        const char *desc;

      if (ipmi_attached)
              return (ENXIO);

        desc = ipmi_pci_match(pci_get_vendor(dev), pci_get_device(dev));
        if (desc != NULL) {
                device_set_desc(dev, desc);
                return (BUS_PROBE_DEFAULT);
        }

        return (ENXIO);
}
```

此函数首先检查全局变量 `ipmi_attached` 的值。如果它不为零，这意味着 `ipmi(4)` 当前正在使用中，将返回错误代码 `ENXIO`；否则，将调用 `ipmi_pci_match` 来确定此驱动程序是否可以处理 `dev`。

## ipmi_pci_match 函数

`ipmi_pci_match` 函数接收一个 PCI 供应商 ID/设备 ID (VID/DID) 对，并验证它是否识别这些 ID。在我定义（随后将逐步介绍）此函数之前，需要描述 `ipmi_identifiers` 数组。此数组在 示例 11-1 的开头附近定义，如下所示：

```
static struct ipmi_ident {
        u_int16_t       vendor;
        u_int16_t       device;
        char            *description;
} ipmi_identifiers[] = {
        { 0x1028, 0x000d, "Dell PE2650 SMIC interface" },
        { 0, 0, 0 }
};
```

如你所见，`ipmi_identifiers` 数组由 `ipmi_ident` 结构组成。每个 `ipmi_ident` 结构包括一个 ![VID/DID](http://atomoreilly.com/source/nostarch/images/1137499.png) ![对](http://atomoreilly.com/source/nostarch/images/1137501.png) 和一个 ![PCI 设备的描述](http://atomoreilly.com/source/nostarch/images/1137503.png)。正如你可能猜到的，`ipmi_identifiers` 列出了 示例 11-1 中第一个 Newbus 驱动支持的设备。

既然我们已经讨论了 `ipmi_identifiers`，让我们来了解一下 `ipmi_pci_match`。

```
const char *
ipmi_pci_match(uint16_t vendor, uint16_t device)
{
        struct ipmi_ident *m;

      for (m = ipmi_identifiers; m->vendor != 0; m++)
                if (m->vendor == vendor && m->device == device)
                        return (m->description);

        return (NULL);
}
```

此函数确定特定的 ![VID/DID](http://atomoreilly.com/source/nostarch/images/1137501.png) 对是否列在 ![ipmi_identifiers](http://atomoreilly.com/source/nostarch/images/1137499.png) 中。如果是，则返回其 ![描述](http://atomoreilly.com/source/nostarch/images/1137503.png)。

## ipmi_pci_attach 函数

`ipmi_pci_attach` 函数是 示例 11-1 中找到的第一个 Newbus 驱动的 `device_attach` 实现。以下是它的函数定义：

```
static int
ipmi_pci_attach(device_t dev)
{
        struct ipmi_softc *sc = device_get_softc(dev);
        struct ipmi_get_info info;
        const char *mode;
        int error, type;

      if (!ipmi_smbios_identify(&info))
                return (ENXIO);

        sc->ipmi_dev = dev;

      switch (info.iface_type) {
        case KCS_MODE:
                mode = "KCS";
                break;
        case SMIC_MODE:
                mode = "SMIC";
                break;
        case BT_MODE:
                device_printf(dev, "BT mode is unsupported\n");
                return (ENXIO);
        default:
                device_printf(dev, "No IPMI interface found\n");
                return (ENXIO);
        }

        device_printf(dev,
            "%s mode found at %s 0x%jx alignment 0x%x on %s\n",
            mode,
            info.io_mode ? "I/O port" : "I/O memory",
            (uintmax_t)info.address,
            info.offset,
            device_get_name(device_get_parent(dev)));

        if (info.io_mode)
              type = SYS_RES_IOPORT;
        else
              type = SYS_RES_MEMORY;

        sc->ipmi_io_rid = PCIR_BAR(0);
        sc->ipmi_io_res[0] = bus_alloc_resource_any(dev, type,
          &sc->ipmi_io_rid, RF_ACTIVE);
        sc->ipmi_io_type = type;
        sc->ipmi_io_spacing = info.offset;

        if (sc->ipmi_io_res[0] == NULL) {
                device_printf(dev, "could not configure PCI I/O resource\n");
                return (ENXIO);
        }

        sc->ipmi_irq_rid = 0;
        sc->ipmi_irq_res = bus_alloc_resource_any(dev, SYS_RES_IRQ,
            &sc->ipmi_irq_rid, RF_SHAREABLE | RF_ACTIVE);

        switch (info.iface_type) {
        case KCS_MODE:
                error = ipmi_kcs_attach(sc);
                if (error)
                        goto bad;
                break;
        case SMIC_MODE:
                error = ipmi_smic_attach(sc);
                if (error)
                        goto bad;
                break;
        }

        error = ipmi_attach(dev);
        if (error)
                goto bad;

        return (0);

bad:
        ipmi_release_resources(dev);
        return (error);
}
```

此函数首先 ![检索](http://atomoreilly.com/source/nostarch/images/1137499.png) 计算机中存储的 IPMI 数据结构，该结构位于 *系统管理 BIOS (SMBIOS)* 中，负责维护硬件配置信息。

根据 SMBIOS 数据，`ipmi_pci_attach` 确定操作模式 `ipmi(4)` 的 ![模式](http://atomoreilly.com/source/nostarch/images/1137501.png) 以及是否需要 ![I/O 端口](http://atomoreilly.com/source/nostarch/images/1137503.png) 或 ![I/O 内存访问](http://atomoreilly.com/source/nostarch/images/1137505.png)。目前，`ipmi(4)` 仅支持键盘控制器样式 (KCS) 和服务器管理接口芯片 (SMIC) 模式。这些模式决定了 IPMI 消息的传输方式。就我们的目的而言，你不需要了解这两种模式的细节。

下一段代码获取 `ipmi(4)` 的 I/O 区域访问权限。在描述此代码之前，需要了解一些关于 PCI 设备的背景信息。启动后，PCI 设备可以将它们的设备寄存器重新映射到不同的位置，从而避免与其他设备发生地址冲突。因此，PCI 设备将它们的 I/O 映射寄存器的大小和当前位置存储在其基本地址寄存器 (BARs) 中。因此，此代码块首先调用 ![PCIR_BAR(0)](http://atomoreilly.com/source/nostarch/images/1137507.png) 以获取第一个 BAR 的地址。然后，它将此地址作为 ![rid](http://atomoreilly.com/source/nostarch/images/1137509.png) 参数传递给 `bus_alloc_resource_any`，从而获取对设备寄存器的 I/O 访问权限。

### 注意

为了准确起见，`PCIR_BAR(x)` 宏返回第 x 个 BAR 的 RID。

`ipmi_pci_attach` 函数的剩余部分获取一个中断请求（IRQ），启动 KCS 或 SMIC 模式，并调用 `ipmi_attach` 以完成设备的初始化。

## `ipmi2_pci_probe` 函数

`ipmi2_pci_probe` 函数是 示例 11-1 中找到的第二个 Newbus 驱动程序的 `device_probe` 实现。以下是其函数定义：

```
static int
ipmi2_pci_probe(device_t dev)
{
        if (pci_get_class(dev) == PCIC_SERIALBUS &&
            pci_get_subclass(dev) == PCIS_SERIALBUS_IPMI) {
                device_set_desc(dev, "IPMI System Interface");
                return (BUS_PROBE_GENERIC);
        }

        return (ENXIO);
}
```

此函数确定 `dev` 是否是 PCI 总线上的通用 IPMI 设备。如果是，则设置其详细描述，并返回成功代码 `BUS_PROBE_GENERIC`。简而言之，此驱动程序处理 PCI 总线上的任何标准 IPMI 设备。

如你所猜，第一个 Newbus 驱动程序是为 Dell PE2650 的一种变通方法（即一种解决方案），因为它不遵循 IPMI 规范。

## `ipmi2_pci_attach` 函数

`ipmi2_pci_attach` 函数是 示例 11-1 中找到的第二个 Newbus 驱动程序的 `device_attach` 实现。以下是其函数定义：

```
static int
ipmi2_pci_attach(device_t dev)
{
        struct ipmi_softc *sc = device_get_softc(dev);
        int error, iface, type;

        sc->ipmi_dev = dev;

      switch (pci_get_progif(dev)) {
        case PCIP_SERIALBUS_IPMI_SMIC:
                iface = SMIC_MODE;
                break;
        case PCIP_SERIALBUS_IPMI_KCS:
                iface = KCS_MODE;
                break;
        case PCIP_SERIALBUS_IPMI_BT:
                device_printf(dev, "BT interface is unsupported\n");
                return (ENXIO);
        default:
                device_printf(dev, "unsupported interface: %d\n",
                    pci_get_progif(dev));
                return (ENXIO);
        }

        sc->ipmi_io_rid = PCIR_BAR(0);
      if (PCI_BAR_IO(pci_read_config(dev, PCIR_BAR(0), 4)))
              type = SYS_RES_IOPORT;
        else
              type = SYS_RES_MEMORY;
        sc->ipmi_io_type = type;
        sc->ipmi_io_spacing = 1;
        sc->ipmi_io_res[0] = bus_alloc_resource_any(dev, type,
            &sc->ipmi_io_rid, RF_ACTIVE);
        if (sc->ipmi_io_res[0] == NULL) {
                device_printf(dev, "could not configure PCI I/O resource\n");
                return (ENXIO);
        }

        sc->ipmi_irq_rid = 0;
        sc->ipmi_irq_res = bus_alloc_resource_any(dev, SYS_RES_IRQ,
            &sc->ipmi_irq_rid, RF_SHAREABLE | RF_ACTIVE);

        switch (iface) {
        case KCS_MODE:
                device_printf(dev, "using KCS interface\n");

                if (!ipmi_kcs_probe_align(sc)) {
                        device_printf(dev,
                            "unable to determine alignment\n");
                        error = ENXIO;
                        goto bad;
                }

                error = ipmi_kcs_attach(sc);
                if (error)
                        goto bad;
                break;
        case SMIC_MODE:
                device_printf(dev, "using SMIC interface\n");

                error = ipmi_smic_attach(sc);
                if (error)
                        goto bad;
                break;
        }

        error = ipmi_attach(dev);
        if (error)
                goto bad;

        return (0);

bad:
        ipmi_release_resources(dev);
        return (error);
}
```

此函数首先检查 `dev` 的编程接口以确定 `ipmi(4)` 的工作模式（SMIC 或 KCS）。然后调用 `PCIR_BAR(0)` 获取第一个 BAR 的地址。从这个 BAR 开始，`ipmi2_pci_attach` 确定在获取之前 `ipmi(4)` 是否需要 I/O 端口或 I/O 内存访问。最后，`ipmi2_pci_attach` 获取一个中断请求，启动 KCS 或 SMIC 模式，并调用 `ipmi_attach` 以完成 `dev` 的初始化。

# 结论

本章检查了 `ipmi(4)` 的 PCI 代码库，并介绍了两个基本概念。首先，一个源文件可以包含多个驱动程序。其次，为了获取 I/O 区域访问权限，PCI 驱动程序必须首先调用 `PCIR_BAR`。
