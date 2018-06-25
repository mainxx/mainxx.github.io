
# error TS2339: Property 'finally' does not exist on type 'Observable\<void\>'

```typescript
save(): void {
        this.saving = true;
        this._addUserRecordsServiceProxy.createOrEdit(this.addUserRecord)
        .finally(() => { this.saving = false; })
        .subscribe(() => {
            this.notify.info(this.l('SavedSuccessfully'));
            this.close();
            this.modalSave.emit(null);
        });
}
```

```bash
ERROR in src/app/main/records/addUserRecords/create-or-edit-addUserRecord-modal.component.ts(52,5): error TS2339: Property 'finally' does not exist on type 'Observable<void>'.
```

解决方案：

```typescript
this._userService.createOrUpdateUser(input)
    .pipe(finalize(() => { this.saving = false; }))
    .subscribe(() => {
        this.notify.info(this.l('SavedSuccessfully'));
        this.close();
        this.modalSave.emit(null);
    });
    //使用.pipe(finalize(() => { this.saving = false; }))
```
