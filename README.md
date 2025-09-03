１ssh ec2-user@54.163.34.250 -i C:\Users\ktc\Downloads\suiyou12gen.pem
２sudo yum install vim -y
３vim ~/.vimrc
set number
set expandtab
set tabstop=2
set shiftwidth=2
set autoindent
4sudo yum install screen -y
5screen
vim ~/.screenrc
hardstatus alwayslastline "%{= bw}%-w%{= wk}%n%t*%{-}%+w"
6 sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
7 sudo mkdir -p /usr/local/lib/docker/cli-plugins/
sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
　docker compose version
8 mkdir dockertest
cd dockertest
9 vim compose.yml
services:
  web:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
       - ./nginx/conf.d/:/etc/nginx/conf.d/
       - ./public/:/var/www/public/
       - image:/var/www/upload/image/
    depends_on:
      - php

  php:
     container_name: php
     build:
       context: .
       target: php
     volumes:
       - ./public/:/var/www/public/
       - image:/var/www/upload/image/

  mysql:
    container_name: mysql
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: example_db
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
      TZ: Asia/Tokyo
    volumes:
      - mysql:/var/lib/mysql
    command: >
      mysqld
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
      --max_allowed_packet=5MB
volumes:
  mysql:
  image:
10 docker compose up
11 mkdir nginx
mkdir nginx/conf.d
12 vim nginx/conf.d/default.conf
 
server {
  listen       0.0.0.0:80;
  server_name  _;
  charset      utf-8;
  client_max_body_size 6M;

  root /var/www/public;

  location ~ \.php$ {
    fastcgi_pass  php:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include       fastcgi_params;
  }

  location /image/ {
      root /var/www/upload;
  }
}
13 mkdir public
14 vim public/zenkikadai.php
<?php
$dbh = new PDO('mysql:host=mysql;dbname=example_db', 'root', '');

if (isset($_POST['body'])) {
  // POSTで送られてくるフォームパラメータ body がある場合

  $image_filename = null;
  if (isset($_FILES['image']) && !empty($_FILES['image']['tmp_name'])) {
    // アップロードされた画像がある場合
    if (preg_match('/^image\//', mime_content_type($_FILES['image']['tmp_name'])) !== 1) {
      header("HTTP/1.1 302 Found");
      header("Location: ./zenkikadai.php");
      return;
    }

    $pathinfo = pathinfo($_FILES['image']['name']);
    $extension = $pathinfo['extension'];
    $image_filename = strval(time()) . bin2hex(random_bytes(25)) . '.' . $extension;
    $filepath =  '/var/www/upload/image/' . $image_filename;
    move_uploaded_file($_FILES['image']['tmp_name'], $filepath);
  }

  // insertする
  $insert_sth = $dbh->prepare("INSERT INTO bbs_entries (body, image_filename) VALUES (:body, :image_filename)");
  $insert_sth->execute([
    ':body' => $_POST['body'],
    ':image_filename' => $image_filename,
  ]);

  // 処理が終わったらリダイレクトする
  header("HTTP/1.1 302 Found");
  header("Location: ./zenkikadai.php");
  return;
}

// いままで保存してきたものを取得
$select_sth = $dbh->prepare('SELECT * FROM bbs_entries ORDER BY created_at DESC');
$select_sth->execute();
?>

<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <title>掲示板</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body {
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif;
      background: #f5f6fa;
      margin: 0; padding: 20px;
      max-width: 760px;
      margin-left: auto; margin-right: auto;
      color: #222;
    }
    h1 { text-align: center; margin-bottom: 20px; }
    form {
      background: #fff;
      padding: 20px;
      border-radius: 14px;
      box-shadow: 0 2px 8px rgba(0,0,0,.08);
      margin-bottom: 20px;
    }
    textarea {
      width: 100%;
      min-height: 100px;
      border: 1px solid #ccc;
      border-radius: 10px;
      padding: 10px;
      font-size: 14px;
      resize: vertical;
    }
    .form-row {
      display: flex;
      gap: 10px;
      align-items: center;
      margin-top: 12px;
    }
    button {
      padding: 10px 16px;
      border: none;
      border-radius: 10px;
      background: #0070f3;
      color: #fff;
      font-weight: bold;
      cursor: pointer;
      transition: .2s;
    }
    button:hover { background: #0059c9; }
    .hint { font-size: 12px; color: #666; margin-top: 6px; }
    .post {
      background: #fff;
      margin-top: 16px;
      padding: 16px;
      border-radius: 14px;
      box-shadow: 0 1px 6px rgba(0,0,0,.05);
    }
    .post .meta { font-size: 12px; color: #999; margin-bottom: 8px; }
    .post .body { white-space: pre-wrap; line-height: 1.6; }
    .post img { max-width: 100%; height: auto; margin-top: 8px; border-radius: 10px; }
  </style>
</head>
<body>
  <h1>掲示板</h1>

  <!-- フォーム -->
  <form method="POST" action="./zenkikadai.php" enctype="multipart/form-data">
    <textarea name="body" required placeholder="投稿内容を入力してください..."></textarea>
    <div class="form-row">
      <input type="file" accept="image/*" name="image" id="imageInput">
      <button type="submit">投稿する</button>
    </div>
    <p class="hint">※ 画像は 5MB 以下、JPEG/PNG/GIF 推奨</p>
  </form>

  <!-- 投稿一覧 -->
  <?php foreach($select_sth as $entry): ?>
    <article class="post">
      <div class="meta">
        #<?= $entry['id'] ?> ｜ <?= $entry['created_at'] ?>
      </div>
      <div class="body">
        <?= nl2br(htmlspecialchars($entry['body'])) ?>
        <?php if(!empty($entry['image_filename'])): ?>
        <div>
          <img src="/image/<?= $entry['image_filename'] ?>" alt="投稿画像">
        </div>
        <?php endif; ?>
      </div>
    </article>
  <?php endforeach; ?>

  <script>
  document.addEventListener("DOMContentLoaded", () => {
    const imageInput = document.getElementById("imageInput");
    imageInput.addEventListener("change", () => {
      if (imageInput.files.length < 1) return;
      if (imageInput.files[0].size > 5 * 1024 * 1024) {
        alert("5MB以下のファイルを選択してください。");
        imageInput.value = "";
      }
    });
  });
  </script>
</body>
</html>
15 vim Dockerfile
FROM php:8.4-fpm-alpine AS php

RUN docker-php-ext-install pdo_mysql

RUN install -o www-data -g www-data -d /var/www/upload/image/

RUN echo -e "post_max_size = 5M\nupload_max_filesize = 5M" >> ${PHP_INI_DIR}/php.ini 
