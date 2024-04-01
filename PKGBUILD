# Maintainer: WithTheBraid <info@braid.business>
# Co-Maintainer: Polarian <polarian@polarian.dev>, Fredy García <frealgagu at gmail dot com>
# Contributor: Philip Goto <philip.goto@gmail.com>

pkgbase=flutter
_group=flutter
groups=("$_group")
pkgver=3.19.5
_dartver=(3.2.0 4.0.0)
_enginever=e76c956498841e1ab458577d3892003e553e4f3c
_materialfontsver=3012db47f3130e62f7cc0beabff968a33cbec8d8
_gradlewver=fd5c1f2c013565a3bea56ada6df9d2b8e96d56aa
_flutterarch=$(uname -m | sed s/aarch64/arm64/ | sed s/x86_64/x64/)
pkgrel=6
pkgdesc="A new mobile app SDK to help developers and designers build modern mobile apps for iOS and Android."
_pkgdesc="Flutter SDK component"
arch=("x86_64" "aarch64")
url="https://${_group}.dev"
license=("custom" "BSD" "CCPL")
makedepends=(
	"dart>=${_dartver[0]}"
	# this breaks using the aur/dart-sdk-dev package
	# "dart<${_dartver[1]}"
	"jq"
	"gradle"
	"unzip"
)
options=("!emptydirs")
source=(
  "${_group}-${pkgver}.tar.xz::https://github.com/${_group}/${_group}/archive/refs/tags/${pkgver/.hotfix/+hotfix}.tar.gz"

  "system-dart.patch"
  "gradle-user-home.patch"

  # thanks to lauren n. liberda from Alpine for the awesome patchset used here !
  "${_group}.sh"
  "version.patch"
  "no-lock.patch"
  "no-runtime-download.patch"
  "doctor.patch"
  "opt-in-analytics.patch"
)

sha256sums=('f9c737cbf6551ca4a68ac826131e1b71f6d173fe83da42521c8f5513c287c57d'
            'd721fc48f534af8f804bb4a9f2cb1d304627a9f73881b3f61d829a9f1e33164f'
            'de0d3567d83bd756841b19ccf879efc02749d8a45cf18d94cd71ec1d366c9024'
            'b4c104129eb57e7e3edca2e23376b8b034de2d466189bdc1c3e2a304506889a3'
            '688a7d6a3c220cf09f7e48af46f1ef1b01d251679962c825eded0b3fa4fc2ab1'
            '544d08716332a9f9358b21010d468b84a9edff0da7bbb1baf0cf4d6322821ea5'
            'a5f19e68e9e4790d017dc4988e715f51c44548df5615aae6106d1a0c84fe49f1'
            '04531ee1732c18c933b5b28f5da88ed183d5aa3698b1d1e912c000928b93ec91'
            '1578e819b6ee479b6db7a095bcfa74372d3ff555642c6d6ea7112e97bb6f2027')

prepare() {
  # this is required in case people try to build with `aur/dart-sdk-dev` instead of `extra/dart`
  DART_BINARY=$(readlink $(which dart))
  DART_ROOT=${DART_ROOT:-${DART_BINARY/\/bin\/dart/}}

  if [ "${DART_ROOT}" != "/opt/dart-sdk" ]; then
    echo -e "\n\n++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\n\n"

    echo -e "  WARNING !!!\n"

    echo -e "  Your default Dart SDK does not seem to be installed into\n"
    echo -e "    /opt/dart-sdk\n"
    echo -e "  Please consider using the original 'extra/dart' package"
    echo -e "  from the Arch Linux package repositories. We otherwise"
    echo -e "  cannot ensure the Flutter tool will work as expected.\n\n"
    echo -e "  Dart executable:	$(which dart)"
    echo -e "  Resolved Dart SDK:	${DART_ROOT}"

    echo -e "\n\n++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\n\n"
  fi

  if [[ ! "$DART_ROOT" =~ ^\/opt\/ && ! "$DART_ROOT" =~ ^\/usr\/ ]]; then
    echo "FATAL: DART_ROOT is neither in /opt nor /usr. You must use a system wide Dart installation for this package. Exiting."
    exit 1
  fi

  if [ -d "${srcdir}/${_group}" ]; then
    rm -rf "${srcdir}/${_group}"
  fi

  mv "${srcdir}/${_group}-${pkgver/.hotfix/+hotfix}" "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/system-dart.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/gradle-user-home.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/version.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/no-lock.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/no-runtime-download.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/doctor.patch" -d "${srcdir}/${_group}"
  patch -p1 -i "${srcdir}/opt-in-analytics.patch" -d "${srcdir}/${_group}"

  echo "${pkgver}" > "${srcdir}/${_group}/version"
  mkdir -p "${srcdir}/${_group}/bin/cache/artifacts"
  cat > "${srcdir}/${_group}/bin/cache/flutter.version.json" <<EOF
{
	"frameworkVersion": "$pkgver",
	"channel": "$_channel",
	"repositoryUrl": "https://github.com/flutter/flutter.git",
	"frameworkRevision": "archlinuxaur0000000000000000000000000000",
	"frameworkCommitDate": "2038-01-19 03:14:08",
	"engineRevision": "$(cat "${srcdir}/${_group}/bin/internal/engine.version")",
	"dartSdkVersion": "$(dart --version | awk '{print $4}')",
	"devToolsVersion": $(jq '.version' < ${DART_ROOT}/bin/resources/devtools/version.json),
	"flutterVersion": "$pkgver"
}
EOF

  if [ -d "${srcdir}/gradlew" ]; then
    rm -rf "${srcdir}/gradlew"
  fi

  mkdir "${srcdir}/gradlew"
  pushd ${srcdir}/gradlew
  gradle init --type basic --project-name flutter --dsl kotlin --incubating
  gradle wrapper
  popd

  cd "${srcdir}/${_group}/bin/cache/artifacts"

  # why should we use a pre-build gradle wrapper if we have it in the arch repos ?
  mkdir -p gradle_wrapper/gradle
  cp -pr "${srcdir}/gradlew/gradle/wrapper" gradle_wrapper/gradle
  cp -pr "${srcdir}/gradlew/gradlew" gradle_wrapper
}

build() {
  export PUB_CACHE="${srcdir}/${_group}/pub-cache"
  cd "${srcdir}/${_group}"
  dart pub get -C "packages/flutter_tools" --no-offline --no-precompile
  dart --verbosity=error --disable-dart-dev \
		--snapshot="bin/cache/flutter_tools.snapshot" --snapshot-kind="app-jit" \
		--packages="packages/flutter_tools/.dart_tool/package_config.json" \
		--no-enable-mirrors "packages/flutter_tools/bin/flutter_tools.dart" --version
 cd ../..

  sed -Ei 's|'"$PUB_CACHE"'|/usr/lib/flutter/pub-cache|g' "${srcdir}/${_group}/packages/flutter_tools/.dart_tool/package_config.json"
  find "$PUB_CACHE" -name '*.aot' -delete
}

_package() {
  pkgdesc="${_pkgdesc} - full installation of development tool and runtime"
  depends=("${pkgbase}-devel=${pkgver}" "${pkgbase}-target-linux=${pkgver}" "${pkgbase}-target-android=${pkgver}" "${pkgbase}-target-web=${pkgver}" "${pkgbase}-intellij-patch=${pkgver}")
  conflicts=("${pkgbase}")
}

_package-common() {
  pkgdesc="${_pkgdesc} - common SDK files and pub cache"
  install="${_group}-common.install"
  conflicts=(
        "${_group}<${pkgver}"
        "${_group}-engine-common<${pkgver}"
        "${_group}-intellij-patch<${pkgver}"
        "${_group}-gradle<${pkgver}"
        "${_group}-engine-android<${pkgver}"
        "${_group}-engine-linux<${pkgver}"
        "${_group}-engine-web<${pkgver}"
        "${_group}-common<${pkgver}"
  )

  install -Dm644 "${srcdir}/${_group}/LICENSE" "${pkgdir}/usr/share/licenses/${_group}/LICENSE"

  install -dm755 "${pkgdir}/usr/lib/${_group}"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_driver"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_goldens"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_localizations"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_test"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_web_plugins"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/fuchsia_remote_debug_protocol"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/integration_test"

  cp -ra "${srcdir}/${_group}/packages/flutter/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter"
  cp -ra "${srcdir}/${_group}/packages/flutter_driver/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter_driver"
  cp -ra "${srcdir}/${_group}/packages/flutter_goldens/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter_goldens"
  cp -ra "${srcdir}/${_group}/packages/flutter_localizations/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter_localizations"
  cp -ra "${srcdir}/${_group}/packages/flutter_test/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter_test"
  cp -ra "${srcdir}/${_group}/packages/flutter_web_plugins/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/flutter_web_plugins"
  cp -ra "${srcdir}/${_group}/packages/fuchsia_remote_debug_protocol/"{pubspec.yaml,lib} "${pkgdir}/usr/lib/${_group}/packages/fuchsia_remote_debug_protocol"
  cp -ra "${srcdir}/${_group}/packages/integration_test/"{pubspec.yaml,lib,android} "${pkgdir}/usr/lib/${_group}/packages/integration_test"
}

_package-target-linux() {
  pkgdesc="${_pkgdesc} - linux target files"
  depends=(
	"${_group}-tool=${pkgver}"
	"${_group}-engine-linux=${pkgver}"
	"dart>=${_dartver[0]}"
	# "dart<${_dartver[1]}"
	"clang"
	"cmake"
        "ninja"
	"pkgconf" # base-devel, but runtime dependency
	# runtime shared libraries
	"gtk3"
	"libglvnd" # https://github.com/flutter/engine/pull/16924
  )

  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/bin"

  cp -ra "${srcdir}/${_group}/packages/flutter_tools/bin/tool_backend.sh" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/bin"
  cp -ra "${srcdir}/${_group}/packages/flutter_tools/bin/tool_backend.dart" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/bin"
}

_package-target-web() {
  pkgdesc="${_pkgdesc} - web target files"
  depends=(
	"${_group}-tool=${pkgver}"
	"${_group}-engine-web=${pkgver}"
  )

  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/lib/src/web"

  cp -ra "${srcdir}/${_group}/packages/flutter_tools/lib/src/web/file_generators" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/lib/src/web"
}

_package-target-android() {
  pkgdesc="${_pkgdesc} - android target files"
  depends=(
	"${_group}-tool=${pkgver}"
	"${_group}-engine-android=${pkgver}"
	"${_group}-gradle=${pkgver}"
  )
  optdepends=("android-sdk: develop for Android devices"
	    "java-environment: develop for Android devices"
  )

  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_tools"
  install -dm755 "${pkgdir}/usr/lib/${_group}/bin/internal"

  cp -ra "${srcdir}/${_group}/packages/flutter_tools/gradle" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools"

  install -Dm644 "${srcdir}/${_group}/bin/internal/engine.version" "${pkgdir}/usr/lib/${_group}/bin/internal"
  install -Dm644 "${srcdir}/${_group}/bin/internal/engine.realm" "${pkgdir}/usr/lib/${_group}/bin/internal"

}

_package-gradle() {
  pkgdesc="${_pkgdesc} - gradle wrapper"
  provides=(
	"${_group}-gradle=${pkgver}"
  )
  conflicts=(
	"${_group}-gradle"
	"${_group}-target-android<${pkgver}"
  )

  install -dm755 "${pkgdir}/usr/lib/${_group}/bin/cache/artifacts"

  cp -ra "${srcdir}/${_group}/bin/cache/artifacts/gradle_wrapper" "${pkgdir}/usr/lib/${_group}/bin/cache/artifacts"
}

_package-tool() {
  pkgdesc="${_pkgdesc} - CLI tool (for packaging only)"
  depends=(
	"${_group}-common=${pkgver}"
	# TODO: completely compile Flutter tool standalone and drop dependency
	"dart>=${_dartver[0]}"
	# "dart<${_dartver[1]}"
	# commands first
	"bash"
	"curl"
	"file" # base-devel, but runtime dependency
	"git"
	"coreutils" # explicit dependency to mkdir, rm
	"unzip"
	"which" # base-devel, but runtime dependency
	"xz"
	"zip"
	"glu" # libGLU.so.1 required for flutter test
  )
  conflicts=("${_group}-devel<${pkgver}" "${_group}-target-android<${pkgver}" "${_group}-target-linux<${pkgver}" "${_group}-target-web<${pkgver}")


  install -dm755 "${pkgdir}/usr/lib/${_group}"
  install -dm755 "${pkgdir}/usr/lib/${_group}/bin/cache"
  install -dm755 "${pkgdir}/usr/lib/${_group}/dev"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/.dart_tool"

  # otherwise flutter analyze will crash
  touch "${pkgdir}/usr/lib/${_group}/dev/.hack-flutter-analyze"

  cp -ra "${srcdir}/${_group}/bin/cache/flutter_tools.snapshot" "${pkgdir}/usr/lib/${_group}/bin/cache/flutter_tools.snapshot"
  cp -ra "${srcdir}/${_group}/bin/cache/flutter.version.json" "${pkgdir}/usr/lib/${_group}/bin/cache"
  cp -ra "${srcdir}/${_group}/version" "${pkgdir}/usr/lib/${_group}"

  cp -ra "${srcdir}/${_group}/packages/flutter_tools/.dart_tool/package_config.json" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools/.dart_tool"

  install -dm755 "${pkgdir}/usr/bin"
  install -Dm755 "${srcdir}/${_group}.sh" "${pkgdir}/usr/lib/${_group}/bin/flutter"
  ln -sf "/usr/lib/flutter/bin/flutter" "${pkgdir}/usr/bin/flutter"
}

_package-devel() {
  pkgdesc="${_pkgdesc} - CLI tool (for application development)"
  depends=(
	"${_group}-tool=${pkgver}"
	"dart>=${_dartver[0]}"
	# "dart<${_dartver[1]}"
  )
  replaces=("${_group}-tool-developer")
  conflicts=("${_group}<${pkgver}")

  install -dm755 "${pkgdir}/usr/lib/${_group}"
  install -dm755 "${pkgdir}/usr/lib/${_group}/packages/flutter_tools"

  cp -ra "${srcdir}/${_group}/examples" "${pkgdir}/usr/lib/${_group}"
  cp -ra "${srcdir}/${_group}/packages/flutter_tools/templates" "${pkgdir}/usr/lib/${_group}/packages/flutter_tools"
  
  # TODO: patch `flutter create` to run without pub cache
  cp -ra "${srcdir}/${_group}/pub-cache" "${pkgdir}/usr/lib/${_group}/pub-cache"
}

_package-intellij-patch() {
  pkgdesc="${_pkgdesc} - IntelliJ Flutter plugin hotfix"
  depends=("${_group}-common=${pkgver}")
  optdepends=(
    "android-studio"
    "intellij-idea-community-edition"
    "intellij-idea-ultimate-edition"
  )

  # this is required in case people try to build with `aur/dart-sdk-dev` instead of `extra/dart`
  DART_BINARY=$(readlink $(which dart))
  DART_ROOT=${DART_ROOT:-${DART_BINARY/\/bin\/dart/}}

  install -dm755 "${pkgdir}/usr/lib/${_group}/bin/cache"
  
  # * not my fault grumble * : The IntelliJ Flutter plugin enforces this relative Dart SDK
  ln -sf "${DART_ROOT}/bin/dart" "${pkgdir}/usr/lib/${_group}/bin/dart"
  ln -sf "${DART_ROOT}" "${pkgdir}/usr/lib/${_group}/bin/cache/dart-sdk"
}

pkgname=("${_group}" "${_group}-common" "${_group}-gradle" "${_group}-tool" "${_group}-devel" "${_group}-target-linux" "${_group}-target-android" "${_group}-target-web" "${_group}-intellij-patch")

for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$_group}")
    _package${_p#$_group}
  }"
done

