<h1 align="center">luci-app-dae</h1>
<p align="center">
  <img width="100" src="https://github.com/daeuniverse/dae/blob/main/logo.png?raw=true" />
</p>
<p align="center">
  <b>A Linux high-performance transparent proxy solution based on eBPF.</b>
</p>

-----------


## Build on OpenWrt official SnapShots

### 1. Get Source

```bash
git clone https://github.com/sbwml/luci-app-dae package/dae
git clone https://github.com/sbwml/v2ray-geodata package/v2ray-geodata
```

### 2. Install clang-13, refer to https://apt.llvm.org

```bash
apt-get update
apt-get install -y clang-13
```

### 3. Change OpenWrt Source (Requirements for DAE to work)

- Enable eBPF support, add content to: `.config`
  ```
  CONFIG_DEVEL=y
  CONFIG_BPF_TOOLCHAIN_HOST=y
  # CONFIG_BPF_TOOLCHAIN_NONE is not set
  CONFIG_KERNEL_BPF_EVENTS=y
  CONFIG_KERNEL_CGROUP_BPF=y
  CONFIG_KERNEL_DEBUG_INFO=y
  CONFIG_KERNEL_DEBUG_INFO_BTF=y
  # CONFIG_KERNEL_DEBUG_INFO_REDUCED is not set
  ```

- Add `xdp-sockets-diag` kernel module, add content to: `package/kernel/linux/modules/netsupport.mk`
  ```mk
  define KernelPackage/xdp-sockets-diag
    SUBMENU:=$(NETWORK_SUPPORT_MENU)
    TITLE:=PF_XDP sockets monitoring interface support for ss utility
    KCONFIG:= \
  	CONFIG_XDP_SOCKETS=y \
  	CONFIG_XDP_SOCKETS_DIAG
    FILES:=$(LINUX_DIR)/net/xdp/xsk_diag.ko
    AUTOLOAD:=$(call AutoLoad,31,xsk_diag)
  endef
  
  define KernelPackage/xdp-sockets-diag/description
   Support for PF_XDP sockets monitoring interface used by the ss tool
  endef
  
  $(eval $(call KernelPackage,xdp-sockets-diag))
  ```

- Patch cgroupfs-mount: use cgroupfs2
  ```bash
  # fix unmount hierarchical mount
  pushd feeds/packages
      curl -s https://raw.githubusercontent.com/sbwml/luci-app-dae/main/.cgroupfs/cgroupfs-mount.init.patch | patch -p1
  popd
  # cgroupfs v2
  mkdir -p feeds/packages/utils/cgroupfs-mount/patches
  curl -s https://raw.githubusercontent.com/sbwml/luci-app-dae/main/.cgroupfs/900-add-cgroupfs2.patch > feeds/packages/utils/cgroupfs-mount/patches/900-add-cgroupfs2.patch
  ```

### 4. Build luci-app-dae

```bash
make menuconfig # choose LUCI -> Applications -> luci-app-dae
make package/dae/luci-app-dae/compile V=s # build luci-app-dae
```

-----------

![dae](https://github.com/sbwml/luci-app-dae/assets/16485166/df81cd85-2957-4bc7-bb57-c561766d7e67)
