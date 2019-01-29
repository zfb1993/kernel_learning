## 路由

- 基本路由  
	- 路由重定向 
	- 视图路由  
  基本路由：比较简单的一种形式
```
	Route::get('foo',function(){
		return 'hello world';
	});
```  
  route/web.php文件定义的路由通过```RouteServiceProvider```被嵌套在一个路由组里。  
  你可以在 ```RouteServiceProvider``` 类中修改此前缀以及其他路由组选项。
- 路由请求方式
```
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```
 有时候需要一个请求响应多种请求方式，可以使用```match```或者```any```方法
```
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('foo', function () {
    //
});
```
- CSRF 保护
 指向web路由定义的```post```,```put```,```delete```路由的任何HTML表单中，
 都应该包含一个CSRF的令牌字段，否则请求将会被拒绝。
```
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```
- 重定向路由
 如果需要重定向到另一个url的路由，可以使用```Route::redirect```方这个方法可以快速的实现重定向，
 而不再需要去定义完整的路由或者控制器:
```
Route::redirect('/here', '/there', 301);

```
- 视图路由
 如果你的路由只需要返回一个视图，可以使用```Route::view```方法。
 不需要定义控制器或者路由。view有三个参数，前两个必填，分别是uri和视图名称，
 第三个参数选填，可以传入一个数组。
```
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```
- 路由参数
1. 必填参数
```
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

2. 选填参数
	
```
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```
3. 正则表达式约束
```
Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```
4. 全局约束
 如果你希望某个具体的路由参数都
 遵循一个正则表达式的约束，就是用```pattern```方法
 在```RouteServiceProvider```的boot方法中定义这些模式：
```
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```
一旦定义好后，便会自动应用这些规则到所有使用该参数名称的路由上：
```
Route::get('user/{id}', function ($id) {
    // 只有在 id 为数字时才执行。
});
```

5. 路由命名
 当为路由指定名称后，就可以用```route```方法，来生成链接或者重定向到该路由：
```
Route::get('user/profile', function () {
    //
})->name('profile');

Route::get('user/profile', 'UserProfileController@show')->name('profile');

```
6. 路由重定向
```
// 生成 URL...
$url = route('profile');

// 生成重定向...
return redirect()->route('profile');
// 传入参数
$url = route('profile', ['id' => 1]);

```
7. 检查当前路由  
 如果想判断当前请求是否指向了某个路由，可以使用named方法进行判断，
```
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```
8. 路由组
