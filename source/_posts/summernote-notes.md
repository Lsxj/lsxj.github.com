---
title: summernote编辑器插件使用笔记
date: 2016-08-10 21:13:01
categories: 前端开发
tags: [javascript, jQuery, jQuery插件,summernote]
---
这次项目中需要用到编辑器插件，于是上网查了一下。由于需要的编辑器功能比较简单，不需要太多复杂功能，所以选择了一款特别轻量的summernote插件，而且后台操作也很简单。
官网：http://summernote.org/
github地址：https://github.com/summernote/summernote

先来看一下官网的截图

![官网截图](http://oao50r2ex.bkt.clouddn.com/image/blog0810_1.png)

麻雀虽小五脏俱全。完全可以满足编辑器的需要。

按照官网链接下载下来的是

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0810_5.png)

我们需要使用的是在dist文件夹内

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0810_2.png)

其中font主要是编辑器内的图标显示，lang是各种语言，css则是样式。我们主要来看一下summernote.js。
## summernote.js
### 定义

     $.fn.extend({
        summernote: function () {
          var type = $.type(list.head(arguments));
          var isExternalAPICalled = type === 'string';
          var hasInitOptions = type === 'object';

          var options = hasInitOptions ? list.head(arguments) : {};

          options = $.extend({}, $.summernote.options, options);
          options.langInfo = $.extend(true, {}, $.summernote.lang['en-US'], $.summernote.lang[options.lang]);

          this.each(function (idx, note) {
            var $note = $(note);
            if (!$note.data('summernote')) {
              var context = new Context($note, options);
              $note.data('summernote', context);
              $note.data('summernote').triggerEvent('init', context.layoutInfo);
            }
          });

          var $note = this.first();
          if ($note.length) {
            var context = $note.data('summernote');
            if (isExternalAPICalled) {
              return context.invoke.apply(context, list.from(arguments));
            } else if (options.focus) {
              context.invoke('editor.focus');
            }
          }
    
          return this;
        }
      });
  
这就是初始化summernote时执行的函数。

    $.extend(object) 可以理解为JQuery 添加一个静态方法。
    $.fn.extend(object) 可以理解为JQuery实例添加一个方法。

默认的options如下

    options: {
      modules: {
        'editor': Editor,
        'clipboard': Clipboard,
        'dropzone': Dropzone,
        'codeview': Codeview,
        'statusbar': Statusbar,
        'fullscreen': Fullscreen,
        'handle': Handle,
        // FIXME: HintPopover must be front of autolink
        //  - Script error about range when Enter key is pressed on hint popover
        'hintPopover': HintPopover,
        'autoLink': AutoLink,
        'autoSync': AutoSync,
        'placeholder': Placeholder,
        'buttons': Buttons,
        'toolbar': Toolbar,
        'linkDialog': LinkDialog,
        'linkPopover': LinkPopover,
        'imageDialog': ImageDialog,
        'imagePopover': ImagePopover,
        'videoDialog': VideoDialog,
        'helpDialog': HelpDialog,
        'airPopover': AirPopover
      },

      buttons: {},
      
      lang: 'zh-CN',

      // toolbar工具栏默认
      toolbar: [
        ['style', ['style']],
        ['font', ['bold', 'underline', 'clear']],
        ['fontname', ['fontname']],
        ['color', ['color']],
        ['para', ['ul', 'ol', 'paragraph']],
        ['table', ['table']],
        ['insert', ['link', 'picture', 'video']],
        ['view', ['fullscreen', 'codeview', 'help']]
      ],

      // popover
      popover: {
        image: [
          ['imagesize', ['imageSize100', 'imageSize50', 'imageSize25']],
          ['float', ['floatLeft', 'floatRight', 'floatNone']],
          ['remove', ['removeMedia']]
        ],
        link: [
          ['link', ['linkDialogShow', 'unlink']]
        ],
        air: [
          ['color', ['color']],
          ['font', ['bold', 'underline', 'clear']],
          ['para', ['ul', 'paragraph']],
          ['table', ['table']],
          ['insert', ['link', 'picture']]
        ]
      },

      // air mode: inline editor
      airMode: false,

      width: null,
      height: null,

      focus: false,
      tabSize: 4,
      styleWithSpan: false,
      shortcuts: true,
      textareaAutoSync: true,
      direction: null,

      styleTags: ['p', 'blockquote', 'pre', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'],

      fontNames: [
        'Arial', 'Arial Black', 'Comic Sans MS', 'Courier New',
        'Helvetica Neue', 'Helvetica', 'Impact', 'Lucida Grande',
        'Tahoma', 'Times New Roman', 'Verdana'
      ],

      fontSizes: ['8', '9', '10', '11', '12', '14', '18', '24', '36'],

      // pallete colors(n x n)
      colors: [
        ['#000000', '#424242', '#636363', '#9C9C94', '#CEC6CE', '#EFEFEF', '#F7F7F7', '#FFFFFF'],
        ['#FF0000', '#FF9C00', '#FFFF00', '#00FF00', '#00FFFF', '#0000FF', '#9C00FF', '#FF00FF'],
        ['#F7C6CE', '#FFE7CE', '#FFEFC6', '#D6EFD6', '#CEDEE7', '#CEE7F7', '#D6D6E7', '#E7D6DE'],
        ['#E79C9C', '#FFC69C', '#FFE79C', '#B5D6A5', '#A5C6CE', '#9CC6EF', '#B5A5D6', '#D6A5BD'],
        ['#E76363', '#F7AD6B', '#FFD663', '#94BD7B', '#73A5AD', '#6BADDE', '#8C7BC6', '#C67BA5'],
        ['#CE0000', '#E79439', '#EFC631', '#6BA54A', '#4A7B8C', '#3984C6', '#634AA5', '#A54A7B'],
        ['#9C0000', '#B56308', '#BD9400', '#397B21', '#104A5A', '#085294', '#311873', '#731842'],
        ['#630000', '#7B3900', '#846300', '#295218', '#083139', '#003163', '#21104A', '#4A1031']
      ],

      lineHeights: ['1.0', '1.2', '1.4', '1.5', '1.6', '1.8', '2.0', '3.0'],

      tableClassName: 'table table-bordered',

      insertTableMaxSize: {
        col: 10,
        row: 10
      },

      dialogsInBody: false,
      dialogsFade: false,

      maximumImageFileSize: null,

      callbacks: {
        onInit: null,//初始化回调函数
        onFocus: null,//聚集
        onBlur: null,//失去焦点
        onEnter: null,//回车键的回调函数
        onKeyup: null,
        onKeydown: null,
        onSubmit: null,//提交时回调函数
        onImageUpload: null,//这就是上传图片的回调函数
        onImageUploadError: null//上传图片出错
      },

      codemirror: {
        mode: 'text/html',
        htmlMode: true,
        lineNumbers: true
      },

      keyMap: {
        pc: {
          'ENTER': 'insertParagraph',
          'CTRL+Z': 'undo',
          'CTRL+Y': 'redo',
          'TAB': 'tab',
          'SHIFT+TAB': 'untab',
          'CTRL+B': 'bold',
          'CTRL+I': 'italic',
          'CTRL+U': 'underline',
          'CTRL+SHIFT+S': 'strikethrough',
          'CTRL+BACKSLASH': 'removeFormat',
          'CTRL+SHIFT+L': 'justifyLeft',
          'CTRL+SHIFT+E': 'justifyCenter',
          'CTRL+SHIFT+R': 'justifyRight',
          'CTRL+SHIFT+J': 'justifyFull',
          'CTRL+SHIFT+NUM7': 'insertUnorderedList',
          'CTRL+SHIFT+NUM8': 'insertOrderedList',
          'CTRL+LEFTBRACKET': 'outdent',
          'CTRL+RIGHTBRACKET': 'indent',
          'CTRL+NUM0': 'formatPara',
          'CTRL+NUM1': 'formatH1',
          'CTRL+NUM2': 'formatH2',
          'CTRL+NUM3': 'formatH3',
          'CTRL+NUM4': 'formatH4',
          'CTRL+NUM5': 'formatH5',
          'CTRL+NUM6': 'formatH6',
          'CTRL+ENTER': 'insertHorizontalRule',
          'CTRL+K': 'linkDialog.show'
        },

        mac: {
          'ENTER': 'insertParagraph',
          'CMD+Z': 'undo',
          'CMD+SHIFT+Z': 'redo',
          'TAB': 'tab',
          'SHIFT+TAB': 'untab',
          'CMD+B': 'bold',
          'CMD+I': 'italic',
          'CMD+U': 'underline',
          'CMD+SHIFT+S': 'strikethrough',
          'CMD+BACKSLASH': 'removeFormat',
          'CMD+SHIFT+L': 'justifyLeft',
          'CMD+SHIFT+E': 'justifyCenter',
          'CMD+SHIFT+R': 'justifyRight',
          'CMD+SHIFT+J': 'justifyFull',
          'CMD+SHIFT+NUM7': 'insertUnorderedList',
          'CMD+SHIFT+NUM8': 'insertOrderedList',
          'CMD+LEFTBRACKET': 'outdent',
          'CMD+RIGHTBRACKET': 'indent',
          'CMD+NUM0': 'formatPara',
          'CMD+NUM1': 'formatH1',
          'CMD+NUM2': 'formatH2',
          'CMD+NUM3': 'formatH3',
          'CMD+NUM4': 'formatH4',
          'CMD+NUM5': 'formatH5',
          'CMD+NUM6': 'formatH6',
          'CMD+ENTER': 'insertHorizontalRule',
          'CMD+K': 'linkDialog.show'
        }
      },
      icons: {
        'align': 'icon-align',
        'alignCenter': 'icon-align-center',
        'alignJustify': 'icon-align-justify',
        'alignLeft': 'icon-align-left',
        'alignRight': 'icon-align-right',
        'indent': 'icon-indent-right',
        'outdent': 'icon-indent-left',
        'arrowsAlt': 'icon-resize-full',
        'bold': 'icon-bold',
        'caret': 'icon-caret-down',
        'circle': 'icon-circle',
        'close': 'icon-close',
        'code': 'icon-code',
        'eraser': 'icon-eraser',
        'font': 'icon-font',
        'frame': 'icon-frame',
        'italic': 'icon-italic',
        'link': 'icon-link',
        'unlink': 'icon-chain-broken',
        'magic': 'icon-magic',
        'menuCheck': 'icon-check',
        'minus': 'icon-minus',
        'orderedlist': 'icon-list-ol',
        'pencil': 'icon-pencil',
        'picture': 'icon-picture',
        'question': 'icon-question',
        'redo': 'icon-redo',
        'square': 'icon-square',
        'strikethrough': 'icon-strikethrough',
        'subscript': 'icon-subscript',
        'superscript': 'icon-superscript',
        'table': 'icon-table',
        'textHeight': 'icon-text-height',
        'trash': 'icon-trash',
        'underline': 'icon-underline',
        'undo': 'icon-undo',
        'unorderedlist': 'icon-list-ul',
        'video': 'icon-facetime-video'
      }
    }
关于编辑器需要的工具栏toolbar具体属性可查看官网[summernote-toolbar属性][1]
### **更改工具栏图标**
由于项目中我是直接使用fontawesome，所以我没有再引入summernote.font，直接在options中的icons中修改。但比较麻烦，不知道有什么更好的方法，求指导。

关于图片上传、提交、按键等回调函数也是在options中，详见[callbacks部分][2]。

初始化一个编辑器很简单。只需要定义
    <div class="summernote" id="myid"></div>
    
     $(function () {
        $('.summernote').summernote();
        //或者
        $('#myid').summernote();
    });

### **设置placeholder:**
    $('.summernote').summernote({
            placeholder:'请输入文章内容',
            ...
        });

### **设置toolbar**
    $('.summernote').summernote({
            toolbar:[
                ['style',['bold','italic','underline','clear']],
                ['fontsize',['fontsize']],
                ['para',['ul','ol','paragraph']],
                ['color',['color']]
            ],
            ...

        });


### **更改图片上传的方式**：
需要提及的是，summernote默认的图片上传方式为base64方式。若需要修改，看以下代码。
【一定要注意：onImageUpload方法修改时要放在callbacks内配置，否则没用】

    $('#myid').summernote({
        callbacks:{
            onImageUpload: function(files, editor, $editable) {
                UploadFiles(files,insertImg);
            }
        },
        ...
    });
    
    function insertImg(){
        for(i in imageUrl){
            $('.summernote').summernote('editor.insertImage',imageUrl[i]);
        }
    }
    
    
    function UploadFiles(files,func){
    //这里files是因为我设置了可上传多张图片，所以需要依次添加到formData中
        var formData = new FormData();
        for(f in files){
            formData.append("file", files[f]);
        }

        $.ajax({
            data: formData,
            type: "POST",
            url: "/uploadMultipleFile",
            cache: false,
            contentType: false,
            processData: false,
            success: function(imageUrl) {
                func(imageUrl);
         
            },
            error: function() {
                console.log("uploadError");
            }
        })
    }
    
我们项目的后台是用spring+springMVC实现的。后台图片上传代码如下:

    @RequestMapping(value = "/uploadMultipleFile", method = RequestMethod.POST, produces = "application/json;charset=utf8")
    @ResponseBody
    public String uploadMultipleFileHandler(@RequestParam("file") MultipartFile[] files, HttpServletRequest request) throws IOException {
        return     UploadUtil.uploadImage(request.getServletContext().getRealPath("/"), files);
    }
    
    
    //UploadUtil.java中uploadImage方法如下
     public static String uploadImage(String serverPath, MultipartFile[] files) {
        try {
            String uploadPath = serverPath + getImageRelativePath();
            String images = "{}";
            //如果不存在目录,创建一个目录
            isDirectory(uploadPath);
            if (files != null && files.length > 0) {
                for (int i = 0; i < files.length; i++) {
                    MultipartFile file = files[i];
                    //save file
                    if (!file.isEmpty()) {
                        String savePath = getImageRelativePath() + file.getOriginalFilename();//数据库保存的图片路径
                        images = JSONUtil.addProperty(images, String.valueOf(i), savePath);
                        save(file, uploadPath);
                    }
                }
            }
            return images;
        } catch (Exception e) {
            e.printStackTrace();
            return "{}";
        }
    }
    
### **设置编辑器中的值：**
    $('#myid').summernote('code',content);

需要注意的是，content是html代码，可能存在**引号嵌套**的问题导致报错，记得将引号进行转义。  
后台处理-java代码：

    content = content.replaceAll("'","\\\\'");
    content = content.replaceAll("\"", "\\\\\"");


### **获取编辑器中的值：**
    var content = $('.summernote').summernote('code');
### **上传附件** 
这次项目需要使用附件，但发现summernote貌似没有附件功能，于是自己研究了一下代码，根据项目的需求，在`link`链接部分进行了修改。
效果如下：

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0810_3.png)
   
首先，我们先看`link`按钮所绑定的事件。

    context.memo('button.link', function () {
            return ui.button({
              contents: ui.icon(options.icons.link),
              tooltip: lang.link.link,
              click: context.createInvokeHandler('linkDialog.show')
            }).render();
          });

由上面的代码可以发现`click`事件为：`linkDialog.show`，那么我们再来看一下`linkDialog`。

    var LinkDialog = function (context) {
        ...
        this.initialize = function () {//初始化
            ...
            var body = '<div class="form-group">' +
                   '<label>' + lang.link.textToDisplay + '</label>' +
                   '<input class="note-link-text form-control" type="text" />' +
                 '</div>' +
                  '<div class="form-group">' +
                  '<label>' + lang.link.attachment + '</label>' +
                  '<input class="note-link-attachment form-control" type="file" />' +
                  '</div>' +
                 '<div class="form-group">' +
                   '<label>' + lang.link.url + '</label>' +
                   '<input class="note-link-url form-control" type="text" value="http://" />' +
                 '</div>' +
                 (!options.disableLinkTarget ?
                   '<div class="checkbox">' +
                     '<label>' + '<input type="checkbox" checked> ' + lang.link.openInNewWindow + '</label>' +
                   '</div>' : ''
                 );
            var footer = '<button href="#" class="btn btn-primary note-link-btn disabled" disabled>' + lang.link.insert + '</button>';
          
        }
    }

    
可以看到，点击链接按钮出现的弹框样式就在`LinkDialog`的`initialize`方法中的`body`，所以我在中间添加了一个`input`上传附件的部分。

    '<div class="form-group">' +
    '<label>' + lang.link.attachment + '</label>' +
    '<input class="note-link-attachment form-control" type="file" />' +
    '</div>' +

那么，我们需要在`lang.link`属性中，新增一个`attachment`附件属性。

![clipboard.png](http://oao50r2ex.bkt.clouddn.com/image/blog0810_4.png)


除此之外，在中文的转换部分summernote-zh-CN.min.js中，添加`link`的`attachment: '添加附件'`

好了，那么我们接下来需要处理的问题是上传文件后的处理。

    this.showLinkDialog = function (linkInfo) {
        return $.Deferred(function (deferred) {
            ...
            //上传文件的输入框
            $linkAttachment = self.$dialog.find('.note-link-attachment'),
            
            ui.onDialogShown(self.$dialog, function () {
                ...
                //对于输入框的事件绑定
                $linkAttachment.on('change', function() {
                    UploadFiles($linkAttachment.val(),function(url){
                        $linkUrl.val(url);//将上传后的URL赋值到linkUrl的输入框中
                    });
                });
            }
        }
    }

UploadFiles与上述修改上传图片的形式一样。   


如果这篇文章对您有帮助，欢迎点赞。如果有疏漏，欢迎指正。

  [1]: http://summernote.org/deep-dive/#custom-toolbar-popover
  [2]: http://summernote.org/deep-dive/#callbacks