#!/bin/bash

# Created from the original package by liolok <aur@liolok.com> and Mike Yuan <me@yhndnzj.com>

# Disable various shellcheck rules that produce false positives in this file.
# Repository rules should be added to the .shellcheckrc file located in the
# repository root directory, see https://github.com/koalaman/shellcheck/wiki
# and https://archiv8.github.io for further information.
# shellcheck disable=SC2034,SC2154
# [ToDo]: Add files: User documentation
# [ToDo]: Add files: Tooling
# [FixMe]: Namcap warnings and errors

# Maintainer: Ross Clark <https://github.com/Archiv8/hexo-cli/discussions>
# Contributor: Ross Clark <https://github.com/Archiv8/hexo-cli/discussions>

pkgname=hexo-cli
pkgver=4.3.0
pkgrel=2
pkgdesc="Command line interface for Hexo"
arch=(
  "any"
)
url="https://www.npmjs.com/package/hexo-cli"
license=(
  "MIT"
)
depends=(
  "nodejs"
)
makedepends=(
  "npm"
  "jq"
)
optdepends=(
  "git: initialize and deploy websites"
  "npm: initialize websites, install plugins"
  "pnpm: npm alternative"
  "yarn: npm alternative"
  "rsync: deploy websites"
)
conflicts=(
  "nodejs-hexo" "nodejs-hexo-cli"
)
replaces=(
  "nodejs-hexo-cli"
)
install=newbie-hint.install
source=(
  https://registry.npmjs.org/${pkgname}/-/${pkgname}-${pkgver}.tgz
)
noextract=(
  "${pkgname}-${pkgver}.tgz"
)
sha512sums=(
  "96be3a875b4ad513502404196f32980161ac9aa1790cbad6eb12846a4aaffe8f49aa075e7a6a418c89bb1ea8dc644501a568a3e043bae57e982620e64fd2d1ee"
)

package() {

  # Ensure cache is set when install is run in order to avoid littering user's home directory
  # ################################################################## #
  printf "\e[1;32m==>\e[0m \e[38;5;248mBuilding nodejs package... \e[0m\n"
  printf "    This may take some time depending on the size of the package and number of dependencies to download... \e[0m\n"

  npm install --global \
    --cache "${srcdir}/npm-cache" \
    --prefix "${pkgdir}"/usr "${srcdir}"/${pkgname}-${pkgver}.tgz

  # Fix incorrect user / group as highlighted by namcap #
  # ################################################### #
  printf "\e[1;32m==>\e[0m \e[38;5;248mCorrecting possible ownership issues\e[0m\n"
  while IFS= read -r fsitem; do
    chown -h root:root "$fsitem"
  done < <(find "${pkgdir}" -not -group root -not -user root)

  # Non-deterministic race in npm gives 777 permissions to random directories.
  # See https://github.com/npm/npm/issues/9359 for details.
  # ######################################################################### #
  printf "\e[1;32m==>\e[0m \e[38;5;248mCorrecting possible permission issues on directories\e[0m\n"
  find "${pkgdir}/usr" -type d -exec chmod 755 "{}" +

  # Remove references to ${pkgdir} #
  # ############################## #
  printf "\e[1;32m==>\e[0m \e[38;5;248mRemoving references to \${pkgdir}\e[0m\n"
  find "${pkgdir}" -type f -name package.json -print0 | xargs -0 sed -i "/_where/d"

  # Remove references to ${srcdir} #
  # ############################## #
  printf "\e[1;32m==>\e[0m \e[38;5;248mRemoving references to \$srcdir \e[0m\n"
  local tmppackage="$(mktemp)"
  local pkgjson="$pkgdir/usr/lib/node_modules/${pkgname}/package.json"
  jq '.|=with_entries(select(.key|test("_.+")|not))' "$pkgjson" > "$tmppackage"
  mv "$tmppackage" "$pkgjson"
  chmod 644 "$pkgjson"

  # Install license #
  # ############### #
  install -dm755 "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s ../../../lib/node_modules/eslint/LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
