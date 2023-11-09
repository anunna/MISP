pkgname=misp
pkgver=2.4.178
pkgrel=1
pkgdesc="Open Source Threat Intelligence and Sharing Platform"
arch=('any')
url="https://github.com/MISP/MISP"
license=('AGPL3')
depends=('git' 'php' 'mariadb')
makedepends=('git')
provides=("misp")
source=("$pkgname"::"git+https://github.com/MISP/MISP.git")
sha512sums=('SKIP')

prepare() {
  cd "$srcdir/$pkgname"
  git checkout tags/v$pkgver
}

build() {
  cd "$srcdir/$pkgname"
  # Install dependencies
  sudo pacman -S --noconfirm coreutils util-linux bash procps-ng diffutils tar xz sed grep gzip filesystem grep gawk
  sudo pacman -S --noconfirm php php-fpm mariadb
  sudo systemctl enable mariadb.service
  sudo systemctl start mariadb.service
  sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
  sudo mysql_secure_installation
  sudo systemctl enable php-fpm.service
  # Clone MISP
  git clone https://github.com/MISP/MISP.git /var/www/MISP
  sudo chown -R http:http /var/www/MISP
  sudo chmod -R 750 /var/www/MISP
  sudo chmod -R g+ws /var/www/MISP/app/tmp
  sudo chmod -R g+ws /var/www/MISP/app/files
  sudo chmod -R g+ws /var/www/MISP/app/files/scripts/tmp
  # Configure database
  sudo mysql -u root -p -e "create database misp; grant usage on *.* to misp@localhost identified by 'misp'; grant all privileges on misp.* to misp@localhost;"
  cd /var/www/MISP/app/Config
  sudo -u http cp -a database.php.default database.php
  sudo -u http sed -i "s/'login' => 'db login'/'login' => 'misp'/g" database.php
  sudo -u http sed -i "s/'password' => 'db password'/'password' => 'misp'/g" database.php
  sudo -u http sed -i "s/'database' => 'db'/'database' => 'misp'/g" database.php
  # Configure MISP
  sudo -u http cp -a core.php.default core.php
  sudo -u http cp -a config.php.default config.php
  sudo -u http echo "CakeResque"
  sudo -u http echo "CakeResque."
  sudo -u http sh -c "echo \"require_once 'database.php';\" >> config.php"
  sudo pear install Crypt_GPG
  sudo pear install Net_GeoIP
  sudo pecl install redis
  echo "extension=redis.so" | sudo tee /etc/php/conf.d/redis.ini
  # Setup workers
  sudo -u http bash -c "nohup sh -c \"export PATH=$PATH:/var/www/MISP/app/Vendor/kamisama/php-resque-ex/lib:/var/www/MISP/app/Vendor/kamisama/php-resque-ex/lib/Resque/; PREFIX=/var/www/MISP/app/Vendor/kamisama/php-resque-ex/lib nohup cake CakeResque.CakeResque tail &\""
  # Reload Apache
  sudo systemctl restart php-fpm.service
}

package() {
  cd "$srcdir/$pkgname"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  cp -r . "$pkgdir/opt/$pkgname"
}

post_install() {
  echo "MISP has been installed under /opt/misp"
  echo "Please configure your web server accordingly"
  echo "For more information, please refer to the MISP documentation"
}
