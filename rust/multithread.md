# スレッド生成


```rs
    // スレッド新規作成
    let handle = std::thread::spawn(|| {
        println!("hoge");
    });

    // スレッドの完了待ち
    handle.join().unwrap();
```



