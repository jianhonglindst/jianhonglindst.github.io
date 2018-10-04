---
layout: post
title:  "python 虛擬環境設定"
date:   2018-05-11 23:30:47 +0800
categories: jekyll update
---

## 建立 virtualenv 及開啟/關閉方式

安裝 

```
$ sudo apt-get python-virtualenv
```

建立虛擬環境 (先`cd`到目標目錄) _.exampleENV_ 是一個例子，可自行設定環境名稱

```
$ virtualenv .exampleENV --python=python3
```
啟動虛擬環境 (先`cd`到建立目錄)

```
$ source .exampleENV/bin/acrivate
```

退出虛擬環境 

```
(.exampleENV) $ deactivate
```

套件庫應該會在 `.exampleENV/bin/pythonX.X/site-packages`

## 確保 Jupyter Notebook 是在虛擬環境執行

#### Method 1: 

先進虛擬環境

```
$ source .exampleENV/bin/activate
```

開啟 Jupyter Notebook

```
(.exampleENV) $ jupyter notebook
```

在頁面上直接選 `NEW > python3` 即可在該虛擬環境中開發

#### Method 2 

建立虛擬環境

```
$ virtualenv .exampleENV --python=python3
```

進入虛擬環境

```
$ source .exampleENV/bin/activate
```

安裝 Jupyter 及 Kernel

```
(.exampleENV) $ pip3 install jupyter
(.exampleENV) $ pip3 install ipykernel
```

新增 Jupyter 上的 Kernel 連結

```
(.exampleENV) $ python3 -m ipykernel install --user --name .exampleENV --display-name "python3(.exampleENV)"
```

開啟 Jupyter Notebook (此時就不一定要在(.exampleENV)環境下開啟Jupyter了)

```
 $ jupyter notebook
```

在頁面上直接選 `NEW > python3(.exampleENV)` 即可在該虛擬環境中開發

## 刪除 jupyter notebook 裡的 kernel

查看現在有何 kernel 在 jupyter 內

```
$ jupyter kernelspec list
```

刪除不需要的 kernel

```
$ jupyter kernelspec remove kernelname
```