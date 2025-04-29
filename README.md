  <div class="info">
                                    <ckeditor [editor]="editor" [config]="editorConfig" formControlName="closeNationalPlanDescription"></ckeditor>
                                </div>

     editorConfig = {
    toolbar: {
      items: [
        'heading', '|',
        'bold', 'italic','|',
        'alignment', '|',
        'numberedList', '|',
        'insertTable', '|',
        'undo', 'redo'
      ]
    }
  };

  HOW TO ADD FOR THIS TEXT EDITOR THE RIGHT TO LEFT DIRECTION IF ARABIC LANGUAGE
