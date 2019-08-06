---
title: CodeIgniterのページネーションクラスとGETパラメータの受け渡し
date: 2019-02-11
categories: ["Misc"]
tags: ["php","codeigniter"]
slug: "ci3pagination"
---
CodeIgniterでpaginationのサンプルを紹介するサイトはいっぱいあるけど、GETパラメータの受け渡しをしたページでページネーションを実装するサンプルが少なかったしうまく行かなかったので、自分のやり方を一例として紹介します。
通常だと`yoursite.com/class/function/ページ` と表示されるのを `yoursite.com/class/function?q=ホニャララ&per_page=ページ`みたいなURLでページネーションを実装します。<br>
環境はPHP7.1 CodeIgniter Version 3.1.0　MariaDB　10.1.33

## 設定ファイルで設定する項目
---

$config['base_url']、$config['total_rows']、$config['per_page']が必須項目。  
`$config['reuse_query_string'] = TRUE;`で?q=ホニャララのクエリストリングを２ページ目以降に引き継ぐことができます。<br><br>
これで試してみると今度はyoursite.com/class/ページfunction?q=ホニャララとなってしまいページネーションがうまく動作しないので、`$config['page_query_string'] = TRUE;`を設定する。<br>この設定で末尾に`&per_page=ページ`のクエリストリングがつくようになるので、`yoursite.com/class/function?q=ホニャララ&per_page=ページ`と表示されるようになります。

ページネーションの設定はこんな感じ

```php
<?php

$this->load->library('pagination');
$config['base_url'] = '/aaa/pagination';
$config['total_rows'] = $num;
$config['per_page'] = $limit;
$config['reuse_query_string'] = TRUE;
$config['page_query_string'] = TRUE;
$this->pagination->initialize($config);
$this->pagination->create_links();

```

## コントローラー
---

```php
<?php

public function pagination()
    {
        $rows = array();
        $limit = 12;
        $offset = $this->input->get('per_page');
        $num = $this->model->num();
        $rows['rows'] = $this->model->page($limit,$offset);
        $rows['pagenation'] = $this->setPage($limit,$num);
        $this->load->view('pagination',$rows);
    }

public function setPage($limit,$num)
    {
        $this->load->library('pagination');
        $config['base_url'] = '/aaa/pagination';
        $config['total_rows'] = $num;
        $config['per_page'] = $limit;
        $config['reuse_query_string'] = TRUE;
        $config['page_query_string'] = TRUE;
        $this->pagination->initialize($config);
        return $this->pagination->create_links();
    }

```

今回は1ページにつき12件取得するのでper_page=は2ページ目以降12,24と変化する。<br><br>
per_page=??は`$this->input->get('per_page')`で受け取ることができるのでこれを`$offset`に代入してsqlのオフセット値にする。
`$limit`はsqlのリミット値と$config['per_page']の値。   

視認性を考慮してページネーションクラスに関する設定はsetPageメソッドに書いています。



## モデル
---

```php
<?php

public function page($limit,$offset)
        {
          $qs = $this->input->get('name'); //取得したGETパラメータ
          $this->db->select('aaa,bbb,ccc');
          $this->db->like('bbb', $qs, 'after');
          $this->db->group_by('ccc');
          $this->db->order_by('bbb','ASC');
          $this->db->limit($limit,$offset);
          $query = $this->db->get('テーブル名');
          return $query->result_array();
        }

public function num()
        {
            $qs = $this->input->get('name');
            $this->db->select('aaa,bbb,ccc');
            $this->db->like('bbb', $qs, 'after');
            $this->db->group_by('ccc');
            $query = $this->db->get('テーブル名');
            return $query->num_rows();
        }

```

numメソッドはトータル件数を取得するメソッド、pageメソッドは取得するデータ。<br>
クエリの書き方は人それぞれ違うけどリミット句とオフセット句は絶対必要なので忘れないでね。

## ビュー
---

```php
  <?php echo $pagenation; ?> //ページネーションを生成
  
  <?php foreach ($rows as $row) :?>
  <?php echo $row['aaa'] ?>
  <?php echo $row['bbb'] ?>
  <?php echo $row['ccc'] ?>
  <?php endforeach ;?>

```

これでページネーションの実装が完成しました。

## ページ番号について
---

通常はページ番号がオフセット値になっているので2ページ目以降が12，24，36のように表示されます。<br>
これを2,3,4と普通のページ番号で表示させたい場合は、<br>
`$config['use_page_numbers'] = TRUE;`をページネーションの設定に加えて、<br>
コントローラーの`$offset = $this->input->get('per_page');`を<br>

```php
<?php
$offset = $this->input->get('per_page') ? ($this->input->get('per_page')-1)*$limit : 0 ;
```
に変更。<br>
これで1ページ目はper_pageがないので0を返し、2ページはper_pageが2なので(2-1)*12で12。そのあとは24、36とオフセット値がセットされます。<br><br>

per_page=をpage=と表示させたい場合は`$config['query_string_segment'] = 'page';`を設定して<br>
コントローラーの$offset = $this->input->get('per_page')を$offset = $this->input->get('page')に変更。