# Cronjob

- [Cronjob](#cronjob)
  - [crontab basic usage](#crontab-basic-usage)

## crontab basic usage

1. `crontab -e ` 打开crontab任务编辑器
2. `* * * * * date >> ~/Desktop/time.txt` 每分钟打印当前时间append到time.txt
3. If you wanna delete a job, just delete the 
