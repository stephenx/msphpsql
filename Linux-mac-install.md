# PHP Linux and Mac Drivers Installation Tutorial
The following instructions assume a clean environment and show how to install PHP 7.x, the Microsoft ODBC driver, Apache, and the Microsoft drivers for PHP for Microsoft SQL Server on Ubuntu 16.04 and 17.10, RedHat 7, Debian 8 and 9, Suse 12, and macOS 10.11 and 10.12. These instructions advise installing the drivers using PECL, but you can also download the prebuilt binaries from the [Microsoft Drivers for PHP for Microsoft SQL Server](https://github.com/Microsoft/msphpsql/releases) Github project page and install them following the instructions in [Loading the Microsoft Drivers for PHP for Microsoft SQL Server](../../connect/php/loading-the-php-sql-driver.md). For an explanation of extension loading and why we do not add the extensions to php.ini, see the section on [loading the drivers](../../connect/php/loading-the-php-sql-driver.md##loading-the-driver-at-php-startup).

These instruction install PHP 7.2 by default -- see the notes at the beginning of each section to install PHP 7.0 or 7.1.

## Contents of this page:

- [Installing the drivers on Ubuntu 16.04 and 17.10](#installing-the-drivers-on-ubuntu-1604-and-1710)
- [Installing the drivers on Red Hat 7](#installing-the-drivers-on-red-hat-7)
- [Installing the drivers on Debian 8 and 9](#installing-the-drivers-on-debian-8-and-9)
- [Installing the drivers on Suse 12](#installing-the-drivers-on-suse-12)
- [Installing the drivers on macOS El Capitan and Sierra](#installing-the-drivers-on-macos-el-capitan-and-sierra)

## Installing the drivers on Ubuntu 16.04 and 17.10

    > [!NOTE]
    > To install PHP 7.0 or 7.1, replace 7.2 with 7.0 or 7.1 in the following commands.

### Step 1. Install PHP
```
sudo su
add-apt-repository ppa:ondrej/php -y
apt-get update
apt-get install php7.2 php7.2-dev php7.2-xml -y --allow-unauthenticated
```
### Step 2. Install prerequisites
Install the ODBC driver for Ubuntu by following the instructions on the [Linux and macOS installation page](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server).

### Step 3. Install the PHP drivers for Microsoft SQL Server
```
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
exit
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
### Step 4. Install Apache and configure driver loading
```
sudo su
apt-get install libapache2-mod-php7.2 apache2
a2dismod mpm_event
a2enmod mpm_prefork
a2enmod php7.2
echo "extension=sqlsrv.so" >> /etc/php/7.2/apache2/php.ini
echo "extension=pdo_sqlsrv.so" >> /etc/php/7.2/apache2/php.ini
```
### Step 5. Restart Apache and test the sample script
```
sudo service apache2 restart
```
To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing the drivers on Red Hat 7

    > [!NOTE]
    > To install PHP 7.0 or 7.1, replace remi-php72 with remi-php70 or remi-php71 respectively in the following commands.

### Step 1. Install PHP

```
sudo su
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
rpm -Uvh remi-release-7.rpm epel-release-latest-7.noarch.rpm
subscription-manager repos --enable=rhel-7-server-optional-rpms
yum-config-manager --enable remi-php72
yum update
yum install php php-pdo php-xml php-pear php-devel re2c gcc-c++ gcc
```
### Step 2. Install prerequisites
Install the ODBC driver for Red Hat 7 by following the instructions on the [Linux and macOS installation page](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md).

Compiling the PHP drivers with PECL with PHP 7.2 requires a more recent GCC than the default:
```
sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
sudo yum install devtoolset-7
scl enable devtoolset-7 bash
```
### Step 3. Install the PHP drivers for Microsoft SQL Server
```
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
exit
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
An issue in PECL may prevent correct installation of the latest version of the drivers even if you have upgraded GCC. To install, download the packages and compile manually:
```
pecl download sqlsrv
tar xvzf sqlsrv-5.2.0.tgz
cd sqlsrv-5.2.0/
phpize
./configure --with-php-config=/usr/bin/php-config
make
sudo make install
```
You can alternatively download the prebuilt binaries from the [Github project page](https://github.com/Microsoft/msphpsql/releases), or install from the Remi repo:
```
sudo yum install php-sqlsrv php-pdo_sqlsrv
```
### Step 4. Install Apache
```
sudo yum install httpd
```
SELinux is installed by default and runs in Enforcing mode. To allow Apache to connect to databases through SELinux, run the following command:
```
sudo setsebool -P httpd_can_network_connect_db 1
```
### Step 5. Restart Apache and test the sample script
```
sudo apachectl restart
```
To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing the drivers on Debian 8 and 9

    > [!NOTE]
    > To install PHP 7.0 or 7.1, replace 7.2 in the following commands with 7.0 or 7.1.

### Step 1. Install PHP
```
sudo su
apt-get install curl apt-transport-https
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
apt-get update
apt-get install –y php7.2 php7.2-dev php7.2-xml
```
### Step 2. Install prerequisites
Install the ODBC driver for Debian by following the instructions on the [Linux and macOS installation page](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). 

You may also need to generate the correct locale to get PHP output to display correctly in a browser. For example, for the en_US UTF-8 locale, run the following commands:
```
sudo su
sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen
```

### Step 3. Install the PHP drivers for Microsoft SQL Server
```
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
exit
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
### Step 4. Install Apache and configure driver loading
```
sudo su
apt-get install libapache2-mod-php7.2 apache2
a2dismod mpm_event
a2enmod mpm_prefork
a2enmod php7.2
echo "extension=sqlsrv.so" >> /etc/php/7.2/apache2/php.ini
echo "extension=pdo_sqlsrv.so" >> /etc/php/7.2/apache2/php.ini
```
### Step 5. Restart Apache and test the sample script
```
sudo service apache2 restart
```
To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing the drivers on Suse 12

    > [!NOTE]
    > To install PHP 7.0, skip the command below adding the repository - 7.0 is the default PHP on suse 12.
    > To install PHP 7.1, replace the repository URL below with the following URL:
      `http://download.opensuse.org/repositories/devel:/languages:/php:/php71/SLE_12/devel:languages:php:php71.repo`

### Step 1. Install PHP
```
sudo su
zypper -n ar -f http://download.opensuse.org/repositories/devel:languages:php/SLE_12/devel:languages:php.repo
zypper --gpg-auto-import-keys refresh
zypper -n install php7 php7-pear php7-devel
```
### Step 2. Install prerequisites
Install the ODBC driver for Suse 12 by following the instructions on the [Linux and macOS installation page](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md).

### Step 3. Install the PHP drivers for Microsoft SQL Server
```
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/sqlsrv.ini
exit
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
### Step 4. Install Apache and configure driver loading
```
sudo su
zypper install apache2 apache2-mod_php7
a2enmod php7
echo "extension=sqlsrv.so" >> /etc/php7/apache2/php.ini
echo "extension=pdo_sqlsrv.so" >> /etc/php7/apache2/php.ini
```
### Step 5. Restart Apache and test the sample script
```
sudo systemctl restart apache2
```
To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing the drivers on macOS El Capitan and Sierra

If you do not already have it, install brew as follows:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

    > [!NOTE]
    > To install PHP 7.0 or 7.1, replace php72 with php70 or php71 respectively in the following commands.

### Step 1. Install PHP

```
brew tap
brew tap homebrew/dupes
brew tap homebrew/versions
brew tap homebrew/homebrew-php
brew install php72 --with-pear --with-httpd24 --with-cgi
echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.bash_profile
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```
### Step 2. Install prerequisites
Install the ODBC driver for macOS by following the instructions on the [Linux and macOS installation page](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). 

In addition, you may need to install the GNU make tools:
```
brew install autoconf automake libtool
```

### Step 3. Install the PHP drivers for Microsoft SQL Server
```
chmod -R ug+w /usr/local/opt/php72/lib/php
pear config-set php_ini /usr/local/etc/php/7.2/php.ini system
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```
### Step 4. Install Apache and configure driver loading
```
(echo "<FilesMatch .php$>"; echo "SetHandler application/x-httpd-php"; echo "</FilesMatch>";) >> /usr/local/etc/httpd/httpd.conf
```
### Step 5. Restart Apache and test the sample script
```
sudo apachectl restart
```
To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Testing Your Installation

To test this sample script, create a file called testsql.php in your system's document root. This is `/var/www/html/` on Ubuntu, Debian, and Redhat, `/srv/www/htdocs` on SUSE, or `/usr/local/var/www` on macOS. Copy the following script to it, replacing the server, database, username, and password as appropriate.
```
<?php
$serverName = "yourServername";
$connectionOptions = array(
    "Database" => "yourDatabase",
    "Uid" => "yourUsername",
    "PWD" => "yourPassword"
);

//Establishes the connection
$conn = sqlsrv_connect($serverName, $connectionOptions);
if( $conn === false ) {
    die( FormatErrors( sqlsrv_errors()));
}

//Select Query
$tsql= "SELECT @@Version as SQL_VERSION";

//Executes the query
$getResults= sqlsrv_query($conn, $tsql);

//Error handling
if ($getResults == FALSE)
    die(FormatErrors(sqlsrv_errors()));
?>

<h1> Results : </h1>

<?php
while ($row = sqlsrv_fetch_array($getResults, SQLSRV_FETCH_ASSOC)) {
    echo ($row['SQL_VERSION']);
    echo ("<br/>");
}

sqlsrv_free_stmt($getResults);

function FormatErrors( $errors )
{
    /* Display errors. */
    echo "Error information: <br/>";
    foreach ( $errors as $error )
    {
        echo "SQLSTATE: ".$error['SQLSTATE']."<br/>";
        echo "Code: ".$error['code']."<br/>";
        echo "Message: ".$error['message']."<br/>";
    }
}
?>
```
Point your browser to http://localhost/testsql.php (http://localhost:8080/testsql.php on macOS). You should now be able to connect to your SQL Server/Azure SQL database.
