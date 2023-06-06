# Laravel開発でのセッション操作の流れ

こんにちは、技術部のMです。<br>
今回は、Laravelで開発するにあたって必須とも言える、**セッション操作**の流れについて自分なりに解説してみたいと思います。

---

## セッションとは
セッションとは、入力したデータなどを一時的に保存できる機能です。<br>
ECサイトのショッピングカート機能など、数ページにわたって情報を保持する場合に使われます。<br>

--- 

## 開発環境
> Linux on x86_64 <br>
> Ubuntu 22.04.2 <br>
> Laravel Framework 10.12.0 <br>
> PHP 8.1.2 <br>
> mysql Ver 8.0.33 
---


## 実例
今回は、新規ユーザー登録を例に使用したいと思います。<br>
「メールアドレス・パスワード入力ページ」と「ユーザー情報入力ページ」で入力した内容をセッションに保存し、保存したデータを「確認ページ」で表示させます。

<img src="img\input-mail-passwd.png">

上記の「メールアドレス・パスワード入力ページ」でメールアドレスとパスワードを入力し、次へボタンを押します。<br>
下記の関数が呼び出され、「ユーザー情報入力ページ」へ遷移されます(ページ遷移の詳細は省略)。<br>
関数内のグローバルセッションヘルパで入力したメールアドレスとパスワードのデータをセッションに保存します。

~~~PHP
    public function userdataPage(Request $request): View
    {
        $email = $request->email;
        $password = $request->password;
        $password_confirmation = $request->password_confirmation;

        //入力したメールアドレスとパスワードをセッションに保存する
        $data = compact('email', 'password', 'password_confirmation');
        session($data);

        return view('auth.input-userdata', $data);
    }
~~~

<img src="img\input-userdata.png">

上記の「ユーザー情報入力ページ」で氏名、郵便番号、住所を入力し、次へボタンを押します。<br>
下記の関数が呼び出され、「確認ページ」へ遷移されます。<br>
関数内で、先ほどセッションに保存したデータを取得します。<br>
入力した氏名、郵便番号、住所のデータをセッションに保存します。<br>
保存したデータを「確認ページ」のビューに渡します。

~~~PHP
    public function confirmUserdataPage(Request $request): View
    {
        //先ほどセッションに保存したデータを取得する
        $session = $request->session()->all();

        $name = $request->name;
        $postal_code = $request->postal_code;
        $address = $request->address;

        $data = compact(
            'name',
            'postal_code',
            'address',
        );

        //名前、郵便番号、住所のデータをセッションに保存する
        session($data);
        
        //セッションに保存したデータを確認ページに渡す
        return view('auth.confirm-userdata', $data);
    }
~~~

渡されたデータをBladeの`{{}}`式で表示します(パスワードは非表示)。

~~~HTML
<form method="POST" action="{{ route('register.store') }}">
        @csrf

        <x-input-label for="email" class="mt-4 border-gray-700" :value="__('Email')" />
        <x-label-item id="email" class="block mb-4 w-full" value="{{ $email }}" />
        <x-input-label for="name" :value="__('Name')" />
        <x-label-item id="name" class="block mb-4 w-full" value="{{ $name }}" />
        <x-input-label for="postal_code" :value="__('Postal Code')" />
        <x-label-item id="postal_code" class="block mb-4 w-full" value="{{ $postal_code }}" />
        <x-input-label for="address" :value="__('Address')" />
        <x-label-item id="address" class="block mb-4 w-full" value="{{ $address }}" />
~~~

数ページにわたって情報が保持できていることを確認できました。

<img src="img\confirm-inputdata.png">
---

最後までお読みいただきありがとうございました！
