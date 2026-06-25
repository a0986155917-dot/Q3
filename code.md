
# 1. 資料庫模型 (models.py)
-介紹：定義圖書館的「借閱紀錄」資料表。包含借閱人、書籍、借出/應還日期，並透過 Python 的 `is_overdue` 計算書籍是否已經過期。
```python
from django.db import models
 from django.contrib.auth.models import User
 from datetime import date

 class BorrowRecord(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="借閱人")
    book = models.ForeignKey('Book', on_delete=models.CASCADE, verbose_name="書籍")
    borrow_date = models.DateField(auto_now_add=True, verbose_name="借出日期")
    due_date = models.DateField(verbose_name="應還日期")
    return_date = models.DateField(null=True, blank=True, verbose_name="實際歸還日期")

    @property
    def is_overdue(self):
        if not self.return_date and date.today() > self.due_date:
            return True
        return False

    def __str__(self):
        return f"{self.user.username} 借了 {self.book.title}"
