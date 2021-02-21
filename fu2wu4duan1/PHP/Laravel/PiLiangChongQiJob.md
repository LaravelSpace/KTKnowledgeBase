```json
{
  "title": "批量重启 Laravel 的 Job",
  "updated_at": "2020-06-18",
  "updated_by": "KelipuTe",
  "tags": "PHP,Laravel,Shell"
}
```

---

## 批量重启 Laravel 的 Job

```shell
#!/bin/bash

cd .../code_path
php artisan queue:restart

sleep 1

reload_queue_name=("job1" "job2" "job3")

for name in ${reload_queue_name[@]};
do
    nohup php artisan queue:work --queue=$name >> /tmp/queue_logs/$name.log &
    echo "$name Job Is Restart..."
done

exit;
```

这段脚本用于 Laravel 项目上线后，批量重启 Job 进程。

首先进入 Laravel 项目根目录，执行 `php artisan queue:restart` 平滑中断所有的 Job 进程。

中间间隔几秒，没啥特殊的作用，单纯的为了把两个操作隔开一段时间。

然后循环执行 `php artisan queue:work` 命令，重启所有的 Job 进程。