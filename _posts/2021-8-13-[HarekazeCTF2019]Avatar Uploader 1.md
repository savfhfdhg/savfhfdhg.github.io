

## 知识点：

1. finfo_file 返回一个文件的信息
2. 请上传小于 256kB 且小于 256px*256px 的 PNG 图片。
3. getimagesize — 取得图像大小





### 题目源码：

```
https://github.com/TeamHarekaze/HarekazeCTF2019-challenges/tree/master/avatar_uploader_1/server
```

### 主要代码

```php
<?php
error_reporting(0);

require_once('config.php');
require_once('lib/util.php');
require_once('lib/session.php');

$session = new SecureClientSession(CLIENT_SESSION_ID, SECRET_KEY);

// check whether file is uploaded
if (!file_exists($_FILES['file']['tmp_name']) || !is_uploaded_file($_FILES['file']['tmp_name'])) {
  error('No file was uploaded.');
}

// check file size
if ($_FILES['file']['size'] > 256000) {
  error('Uploaded file is too large.');
}

// check file type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);
if (!in_array($type, ['image/png'])) {
  error('Uploaded file is not PNG format.');
}

// check file width/height
$size = getimagesize($_FILES['file']['tmp_name']);
if ($size[0] > 256 || $size[1] > 256) {
  error('Uploaded image is too large.');
}
if ($size[2] !== IMAGETYPE_PNG) {
  // I hope this never happens...
  error('What happened...? OK, the flag for part 1 is: <code>' . getenv('FLAG1') . '</code>');
}

// ok
$filename = bin2hex(random_bytes(4)) . '.png';
move_uploaded_file($_FILES['file']['tmp_name'], UPLOAD_DIR . '/' . $filename);

$session->set('avatar', $filename);
flash('info', 'Your avatar has been successfully updated!');
redirect('/');
```

### finfo_file

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);
if (!in_array($type, ['image/png'])) {
  error('Uploaded file is not PNG format.');
}
finfo_file获取文件的信息，图片类型必须是image/png
```

### getimagesize

```php
$size = getimagesize($_FILES['file']['tmp_name']);
if ($size[0] > 256 || $size[1] > 256) {
  error('Uploaded image is too large.');
}
if ($size[2] !== IMAGETYPE_PNG) {
  // I hope this never happens...
  error('What happened...? OK, the flag for part 1 is: <code>' . getenv('FLAG1') . '</code>');
}
getimagesize取得图像大小，$size[0]，$size[1]都小于256
$size[2] !== IMAGETYPE_PNG 意思是内容不能是png图片
```

### #flag获取

生成一个正常的图片，

保留文件头，将其与多余部分删除，保存编辑好的png图片

```php
塒NG

   
IHDR
```

```php
What happened...? OK, the flag for part 1 is: flag{b1dfbc41-46b0-4f19-8343-4377c9700bc6}
```

