---
layout: post
title: laravel 异步
tags: ['laravel']
---

## laravel的异步

### 5.0版本可以直接。

<pre>
	$process = new Process('cd ' . base_path() . ' && php 	artisan migrate --seed');
	$process->run();
</pre>

### 4.2的有以下的解决方案

1. controller中调用。不行

<pre>
	Artisan::call('card:tools',[
    	'--method'=>'modifyBatch',
    	'--batchId'=>$data['batchId'],
    	'--batchExpireDate'=>$data['batchExpireDate'],
    	'--mutex'=>$data['mutex'],
	]);

</pre>

2. 使用SSH 包 可行，但是要配置 remote.php

<pre>
	SSH::run( array(
    	'cd ../' .app_path(),
    	'card:tools --method=modifyBatch --batchId='.$data[ 'batchId'].' --batchExpireDate='. $data['batchExpireDate']. ' --mutex='.$data[ 'mutex'],
	));
</pre>

3. 最佳方案，还是 popen

<pre>
	$phpBin=PHP_BINDIR.'/php';
	$artisan=base_path().'/artisan';
	$command='card:tools --method=modifyBatch --batchId='.$actData[ 'batchId'].' --batchExpireDate='. $actData['batchExpireDate']. ' --mutex='.$actData[ 'mutex'].'';
	pclose( popen($phpBin.' '.$artisan.' '.$command .' &','r'));
</pre>


