# Maintainer: Sven-Hendrik Haase <sh@lutzhaase.com>
# Co-Maintainer: Konstantin Gizdov <arch@kge.pw>
pkgbase=cuda
pkgname=(cuda nvidia-nsight)
#pkgname+=(libcudart)
pkgver=10.1.168
_basever=10.1.105
_driverver=418.39
pkgrel=9
pkgdesc="NVIDIA's GPU programming toolkit"
arch=('x86_64')
url="https://developer.nvidia.com/cuda-zone"
license=('custom:NVIDIA')
options=(!strip staticlibs)
install=cuda.install
source=(https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_${_basever}_${_driverver}_linux.run
        https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.168_418.67_linux.run
        cuda.sh
        cuda.conf
        cuda-findgllib_mk.diff)
sha512sums=('07cdc053989758e4cdccaad566cae7127fa79780ad85033ea52d33cc4c43c9de08e88dd087390c58b542e51703f256859cdf51a018ee57e6a412f45fb9561622'
            '85635012ec7081e200fb46f973852939f9709e802f495c288d5f31ff1fbd283a578fdaca0afa8955e6e0c7ca6c4862428da5168e0938b48390952d8633350910'
            'ce0b8df5d918ec8429da4ab8f9dee463ac04055ee5b0beeb0386b67d765a4c892d314995776a0d695cd06bcfbaf996e4904935ddc898a9d774f6bf965d989dea'
            '714d973bc79446f73bebe85306b3566fe25b554bcbcba2fcbe76709a3eca71fb5d183ab4da2d3b5e9326cb9cd8d13a93f6d4a005ea5a41f7ef8e6c6e81e06b5e'
            '41d6b6cad934f135eafde610d1cbd862033977fd4416a4b6abaa47709a70bab7fcf6f8377c21329084fb9db13f2a8c8c20e93c15292d7d4a6448d70a33b23f1b')

prepare() {
  sh cuda_${_basever}_${_driverver}_linux.run --target "${srcdir}" --noexec
  sh cuda_10.1.168_418.67_linux.run --target "${srcdir}" --noexec

  # Fix up samples tht use findgllib_mk
  for f in builds/cuda-samples/*/*/findgllib.mk; do
    patch $f cuda-findgllib_mk.diff
  done
}

package_cuda() {
depends=('gcc8-libs'  'gcc8' 'opencl-nvidia' 'nvidia-utils')
optdepends=('nvidia-nsight: for Nvidia IDE, examples, samples, doc, nvpp')
replaces=('cuda-toolkit' 'cuda-sdk')
provides=('cuda-toolkit' 'cuda-sdk')
  mkdir -p "${pkgdir}/opt/"

  cd "${srcdir}/builds"
  cp -r cuda-toolkit "${pkgdir}/opt/cuda"
  cp -r cublas/include/* "${pkgdir}/opt/cuda/include/"
  cp -r cublas/lib64/* "${pkgdir}/opt/cuda/lib64/"
  cp -r cuda-samples "${pkgdir}/opt/cuda/samples"
  ln -s /opt/cuda/targets/x86_64-linux/lib "${pkgdir}/opt/cuda/lib"
  ln -s /opt/cuda/nvvm/lib64 "${pkgdir}/opt/cuda/nvvm/lib"

  # Define compilers for CUDA to use.
  # This allows us to use older versions of GCC if we have to.
  # Allow ccache to work with CUDA compiler.
  echo -e "#!/bin/sh -\n[ -f /usr/bin/ccache ] && exec /usr/bin/ccache /usr/bin/gcc-8 \"\$@\" || exec /usr/bin/gcc-8 \"\$@\"" > "${pkgdir}/opt/cuda/bin/gcc"
  echo -e "#!/bin/sh -\n[ -f /usr/bin/ccache ] && exec /usr/bin/ccache /usr/bin/g++-8 \"\$@\" || exec /usr/bin/g++-8 \"\$@\"" > "${pkgdir}/opt/cuda/bin/g++"
  chmod +x ${pkgdir}/opt/cuda/bin/g{cc,++}
# ln -s /usr/bin/gcc-8 "${pkgdir}/opt/cuda/bin/gcc"
# ln -s /usr/bin/g++-8 "${pkgdir}/opt/cuda/bin/g++"
  
  # Allow ccache to work with NVCC
  mv "${pkgdir}/opt/cuda/bin/nvcc" "${pkgdir}/opt/cuda/bin/nvcc.bin"
  echo -e "#!/bin/sh -\n[ -f /usr/bin/ccache ] && exec /usr/bin/ccache /opt/cuda/bin/nvcc.bin \"\$@\" || exec /opt/cuda/bin/nvcc.bin \"\$@\"" > "${pkgdir}/opt/cuda/bin/nvcc"
  chmod +x "${pkgdir}/opt/cuda/bin/nvcc"

  # Create soname links.
  # We have to be weird about this since for some reason the ELF SONAME is incorrect or at least partially incorrect for some libs.
  # Best we can do is create all symlinks and hope for the best.
  # Their installer used to perform this for us but now it's all manual and I think this is what we'll be stuck with for now.
  find cuda-toolkit/targets -type f -name '*.so*' ! -path '*stubs/*' -print0 | while read -rd $'\0' _lib; do
    _base=${_lib%.so.*}
    _current_soname=$(basename ${_lib%.*})
    while [[ $_current_soname != $(basename $_base) ]]; do
      ln -sf ${_lib##*/} ${pkgdir}/opt/cuda/lib64/$_current_soname
      _current_soname=${_current_soname%.*}
    done
  done

  # Install profile and ld.so.config files
  install -Dm755 "${srcdir}/cuda.sh" "${pkgdir}/etc/profile.d/cuda.sh"
  install -Dm644 "${srcdir}/cuda.conf" "${pkgdir}/etc/ld.so.conf.d/cuda.conf"

  mkdir -p "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s /opt/cuda/doc/pdf/EULA.pdf "${pkgdir}/usr/share/licenses/${pkgname}/EULA.pdf"

  # Remove included copy of java
  rm -fr  "${pkgdir}/opt/cuda/jre"

  # Allow GCC 9 to work
  sed -i "/.*unsupported GNU version.*/d" "${pkgdir}"/opt/cuda/targets/x86_64-linux/include/crt/host_config.h

  # Remove nsight, extra etc.
  local targets=('Nsight{Compute,Systems}*' 'libnsight' 'nsightee_plugins' 'doc' 'samples' 'extras' 'libnvvp')
  for target in ${targets[@]}; do echo "$target"; eval rm -rf ${pkgdir}/opt/cuda/$target; done
# xargs -I{} rm -rf ${pkgdir}/opt/cuda/{} <(echo ${targets[@]})
}

package_nvidia-nsight() {
depends=('cuda' 'gdb' 'java-runtime=8')
  mkdir -p "${pkgdir}/opt/cuda/"

  cd "${srcdir}/builds/cuda-toolkit"
  local targets=('Nsight{Compute,Systems}*' 'libnsight' 'nsightee_plugins' 'doc' 'extras' 'libnvvp')
  eval cp -r ${targets[@]} -t "${pkgdir}/opt/cuda"
  cp -r ../cuda-samples "${pkgdir}/opt/cuda/samples"

  # Link to system java 8
  sed 's|../jre/bin/java|/usr/lib/jvm/java-8-openjdk/jre/bin/java|g' \
    -i "${pkgdir}/opt/cuda/libnsight/nsight.ini" \
    -i "${pkgdir}/opt/cuda/libnvvp/nvvp.ini"

}

# vim:set ts=2 sw=2 et:

