---
layout: post
title:  "Install R In Ubuntu And Upgrade Last Version Of R"
date:   2017-12-29 21:30:47 +0800
categories: jekyll update
---

我是個 Linux 新手，所以只是跟著網路上的教學操作，很多步驟的含義我並不清楚，本文僅紀錄我安裝 `R` 及更新 `R版本` 的過程，盡量將我遇到的狀況完整紀錄，以供自己回顧及有需要的人參考，如果想知道較詳細的說明，請直接看參考資訊的連結。

Ubuntu 環境資訊
---------------

    Distributor ID: Ubuntu
    Description:    Ubuntu 16.04.3 LTS
    Release:        16.04
    Codename:       xenial

本文內容
--------

-   安裝 OpenJDK 環境
-   安裝 Orcale Java JDK 環境
-   設定 Java 版本
-   安裝 R
-   更新 R
-   安裝 IDE
-   參考資訊

進入終端機

    Ctrl + Alt + T 

安裝 OpenJDK 環境
-----------------

使用 `apt` 安裝

    sudo apt-get update 

安裝 `JRE` : 執行一般 `Java` 應用程式

    sudo apt-get install default-jre 

安裝 `JDK` : 執行 `Java` 開發程式

    sudo apt-get update install default-jdk 

檢查 `Java` 版本

    java -version
    javac -version

安裝 Orcale Java JDK 環境
-------------------------

新增 `PPA`

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update 

選擇需要的 `Java-JDK` 版本安裝: 雖然 `java9` 已發佈，但我不知道對於 `R` 的開發會不會有問題，所以這邊我選擇 `java8`

    sudo apt-get install oracle-java8-installer 

安裝到這邊，畫面會出現 `Oracle` 的授權條款，要同意才能安裝，所以直接 `<Yes>` &gt;&gt; `<Yes>`

設定 Java 版本
--------------

列出系統預設及所有可用的 `Java` 版本

    update-alternatives --query java 

會看到像這樣的畫面

![](https://jianhonglindst.github.io/assets/2017-12-29-install-R-in-ubuntu-with-upgrade-last-version/0001_update-alternatives%20--query%20java.png)

設定 `JAVA_HOME` 環境變數: 查詢 `Java`安裝路徑並加入路徑設定

    JAVA_HOME="/usr/lib/jvm/java-8-oracle"
    source /etc/environment
    echo $JAVA_HOME

到這邊，完成了開發環境需要的建置，接著要安裝 `R`

安裝 R
------

安裝 `R`

    sudo apt-get update
    sudo apt-get install -y --allow-unauthenticated r-base

安裝環境套件

    sudo apt-get install -y libssl-dev
    sudo apt-get install -y libcurl4-openssl-dev

設定 `R`

    sudo R CMD javareconf 

到這邊就可以在終端機中叫出 `R` 來使用了！！

![fa-rotate-90](https://jianhonglindst.github.io/assets/2017-12-29-install-R-in-ubuntu-with-upgrade-last-version/0002_TerminalR.png) (迷..) 我無法轉動它...

> 有時安裝 `install.packages()` 時會有缺乏環境套件而失敗的問題，此時 `R Console` 會跳出類似 `error: no "lssl-dev"` 之類的訊息，此時這邊的 `l` 後面所接的 `xxx`就代表電腦未安裝的套件，以此例 `xxx` = `ssl`，所以我們就必須在終端機上安裝 `libssl-dev`，方法為輸入指令 `sudo apt-get install -y libssl-dev`。這個方式可以解掉蠻多套件不能安裝的問題，如果有其他情形就不在此限摟～

更新 R
------

修改 `sources.list`

    sudo gedit /etc/apt/sources.list

在 `sources.list` 這個檔案最下面新增下列命令 (適用於 ubuntu 16.04 LTS, 其他版本會不同，可參照**參考資訊**)

    deb https://cloud.r-project.org//bin/linux/ubuntu xenial/

`R` 的 `CRAN` 中儲存的 `ubuntu` 套件需要通過驗證，用以下指令:

    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

最後再進行更新

    sudo apt-get update
    sudo apt-get upgrade

這樣就可以成功的更新到最新版的 `R` 囉。

安裝 IDE
--------

IDE 我選擇使用 RStudio，下載連結在此 [RStudio 下載網址](https://www.rstudio.com/products/rstudio/download/#download) 。

下載以後點執行檔兩下即可安裝。

我的下載檔名如右 `rstudio-xenial-1.1.383-amd64.deb`

參考資訊
--------

1.  [Ubuntu Linux 安裝 Oracle 或 OpenJDK 的 Java JRE 與 JDK 步驟教學](https://blog.gtwang.org/linux/how-to-install-java-with-apt-get-on-ubuntu-linux/)
2.  [正確的更新 R](http://www.linuxdiyf.com/linux/22014.html)
3.  [RStudio](https://www.rstudio.com/)
