# Compile

- Install dependence modules
  - Install tools

    ```
    (CentOS) sudo yum install gcc gcc-c++ autoconf automake libtool pkgconfig cppunit-devel
    (Ubuntu) sudo apt-get install build-essential autoconf automake libtool libcppunit-dev
    (OSX)    brew install autoconf automake libtool pkg-config cppunit
    ```
  
  - Install libevent
  
    ```
    $ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
    $ tar xfz libevent-2.0.21-stable.tar.gz
    $ pushd libevent-2.0.21-stable
    $ ./configure
    $ make
    $ sudo make install
    $ popd
    ```
  - Install [arcus-zookeeper](https://github.com/naver/arcus-zookeeper)

## Test

- Method of performing individual tests 
  
    ```
    perl t/${test_script_name}.t
    ```
    
- If several tests were performed at the same time and used port fails, then re-execute or set synchronization to 1 and perform 

    ```
    //run_test.pl 수정
    my $opt = '--job 1'; # --job N : run N test jobs in parallel
    ```

- Failed due to perl Test::More.pm module does not exist 
    
    ```
     Can't locate Test/More.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl
     /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at -e line 1.
    ```

- If you get above mention error, then reason is the perl module Test::More.pm required by the test script
  is not installed on the system. Using CPAN install the Test::More.pm module.

    ```
     # Install CPAN
     (CentOS) sudo yum install cpan
     (Ubuntu) sudo apt-get install libpath-tiny-perl

     # Install Test::More
     cpan Test::More
    ```
