## quill上传图片组件修改

> 富文本编辑器quill在上传图片的时候会转换为base64格式，直接插入到content，这样后端直接存图片而不是图片地址了，需要修改为存图片地址的方式

> 在查看github之后发现可以增加一个自定义toolbar来解决


1. 增加自定义的toolbar

```

 <quill-editor (onEditorCreated)="EditorCreated($event)" [(ngModel)]="data['noticeContent']" formControlName="noticeContent">
                <div quill-editor-toolbar>
                    
                    <span class="ql-formats">

                        <button nz-button type="button" (click)="customButtonClick($event)" class="ql-image">imgage</button>
                        <input class="open-file" type="file" name="file" id="file" 
                        accept="image/png, image/gif, image/jpeg, image/bmp, image/x-icon"
                        style="display: none;" (change)="upload($event)" multiple="false"
                        />
                    </span>
                 

                </div>
            </quill-editor>

```

2. 编写customButtonClick和upload方法

```

  EditorCreated(quill) {
    this.editor = quill;
  }

  customButtonClick(quill){

    var range = this.editor.getSelection(true);
    var  length = range.index;
	//弹出文件选择框调用upload方法
    this.el.nativeElement.querySelector('.open-file').click();

  }
  

  uploadImg(){

    this.file = [];
    let imgFile=window.document.getElementById('file')['files'][0];
    if(imgFile==null){
      return;
    }
    this.file.push(imgFile);

    if (this.file && this.file !== null && this.file.length>=1) {
	 //上传图片
      this.service.imageUpload(this.file).subscribe(res => {
        if (res['body']) {
          if (res['body']['resCode'] === 20000) {
            let imgUrl=res['body']['data'][0]
            if(imgUrl!=null){
              this.editor.insertEmbed(length, 'image',imgUrl);
            }

          }else {
            this.message.create('error', res['body']['resCode']);
        }
      }
    })

  }
}

```

3. 参考资料
> 