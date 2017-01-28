### 假如iPhone丢失／AppleID被盗／不慎删除了联系人 如何找回通讯录？

如果以前在电脑上用iTune备份过的话可以从备份文件中恢复通讯录
mac 上备份文件存在~/Library/Application Support/MobileSync/Backup
通讯录的文件名为：
`31bb7ba8914766d4ba40d6dfb6113c8b614be442`
此文件为SQLite格式
可以使用 GUI tool http://sqlitebrowser.org/ 打开
以下SQL 得到姓名+电话
```SQL
select b.last || b.first as ContactName, a.value as PhoneNumber  
  from abMultiValue a, abperson b 
 where a.record_id = b.rowid 
 order by b.last 
```