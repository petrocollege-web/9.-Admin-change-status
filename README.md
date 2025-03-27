### Быстрое изменение статуса заявки администратором

#### 1. Добавляем кнопки в представление (view)

**В лекции используются идентификаторы статусов:** 
![image](https://github.com/user-attachments/assets/57418d98-81c5-4644-92ab-2204aed49933)



Модифицируем `views/request/view.php`:

```php
<?= DetailView::widget([...]) ?>

<?php if (Yii::$app->user->identity->role_id === 2): ?>
<div class="card mt-4">
    <div class="card-header bg-primary text-white">
        <h5>Изменение статуса</h5>
    </div>
    <div class="card-body">
        <div class="btn-group" role="group">
            <?= Html::a('В работу', ['change-status', 'id' => $model->id, 'status' => 2], // status => 2 - Id статуса "В работе"
            [
                'class' => 'btn btn-warning',
                'data' => ['method' => 'post']
            ]) ?>
            
            <?= Html::a('Выполнено', ['change-status', 'id' => $model->id, 'status' => 4], [
                'class' => 'btn btn-success',
                'data' => ['method' => 'post']
            ]) ?>
            
            <?= Html::button('Отменить', [
                'class' => 'btn btn-danger',
                'data-bs-toggle' => 'modal',
                'data-bs-target' => '#cancelModal'
            ]) ?>
        </div>
    </div>
</div>

<!-- Модальное окно для причины отмены -->
<div class="modal fade" id="cancelModal" tabindex="-1">
    <div class="modal-dialog">
        <?= Html::beginForm(['change-status', 'id' => $model->id], 'post') ?>
        <div class="modal-content">
            <div class="modal-header bg-danger text-white">
                <h5 class="modal-title">Укажите причину отмены</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
            </div>
            <div class="modal-body">
                <?= Html::hiddenInput('status', 'отменено') ?>
                <?= Html::textarea('cancel_reason', '', [
                    'class' => 'form-control',
                    'rows' => 3,
                    'required' => true
                ]) ?>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Закрыть</button>
                <button type="submit" class="btn btn-danger">Подтвердить отмену</button>
            </div>
        </div>
        <?= Html::endForm() ?>
    </div>
</div>
<?php endif; ?>
```

![image](https://github.com/user-attachments/assets/ecfad1b4-f896-4e1c-ad3e-a130a27e49fc)
![image](https://github.com/user-attachments/assets/b136e19c-eb6f-48e5-adff-06bb3241839f)

---

#### 2. Создаем действие в контроллере

Добавляем в `controllers/RequestController.php`:

```php
public function actionChangeStatus($id, $status = 3) // 3 - Id статуса отмены
    {
        $model = $this->findModel($id);
        
        $model->status_id = $status;
        $model->cancel_reason = Yii::$app->request->post('cancel_reason');

        if ($model->save()) {
            
            Yii::$app->session->setFlash('success', 'Статус успешно изменен');
        } else {
            Yii::$app->session->setFlash('error', 'Ошибка при изменении статуса');
        }
        
        return $this->redirect(['view', 'id' => $id]);
    }
```



---

#### 4. Стилизация (по желанию)

Добавляем в `web/css/site.css`:

```css
/* Анимация изменения статуса */
.status-changing {
    transition: all 0.3s ease;
    transform: scale(1.05);
}
```

---

### Итоговый результат:

1. **Только для администратора**  отображаются кнопки изменения статуса
2. **Три основных статуса**:
   - "В работу" (желтая кнопка)
   - "Выполнено" (зеленая кнопка)
   - "Отменить" (красная кнопка с модальным окном для причины)
