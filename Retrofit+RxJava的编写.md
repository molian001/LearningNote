
Retrofit是目前最火的android网络框架，而Rxjava（Rxandroid）官方介绍是"a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。

简而言之Rxjava是一个用观察者模式实现异步操作的库,由于其语法逻辑直观简洁，用线性结构展示异步操作，不用进行各种复杂的回调操作（内部封装好了），现在运用越来越广泛。

两者结合会让网络请求逻辑清晰直观
<!--more-->



##retrofit
- 官方地址：http://square.github.io/retrofit/

###retrofit的基本使用
retrofit的使用首先需要构建一个定义网络请求方法的接口

	public interface RetrofitService{
		 /**
         * 登录
         * @param loginRequest
         * @return
         */
        @POST("user/login")
        Call<APIResponse<>> login(@Body LoginRequest loginRequest);//官方格式
    }
    
    //根据服务端定义的接口所返回的数据定义该类
    public class LoginResponse{
    	
        private User user;
        private Token token;
    
        public User getUser() {
            return user;
        }
    
        public void setUser(User user) {
            this.user = user;
        }
    
        public Token getToken() {
            return token;
        }
    
        public void setToken(Token token) {
            this.token = token;
        }
    }
    //定义通用的服务器返回格式：一个错误码+泛型Date
    public class APIResponse<T> extends BaseModel {
    private int code = -2;
    private T data;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
	}


    public class User  {
        private long id;
        private String mobile;
        private String email;
        private String nickname;
        private String avatar;
    
        public long getId() {
            return id;
        }
    
        public void setId(long id) {
            this.id = id;
        }
    
        public String getMobile() {
            return mobile;
        }
    
        public void setMobile(String mobile) {
            this.mobile = mobile;
        }
    
        public String getEmail() {
            return email;
        }
    
        public void setEmail(String email) {
            this.email = email;
        }
    
        public String getNickname() {
            return nickname;
        }
    
        public void setNickname(String nickname) {
            this.nickname = nickname;
        }
    
        public String getAvatar() {
            return avatar;
        }
    
        public void setAvatar(String avatar) {
            this.avatar = avatar;
        }
	}
	
	//根据服务端接口所需要的数据定义该类
	public class LoginRequest{
    private String mobile;
    private String password;

    public LoginRequest(String mobile, String password) {
        this.mobile = mobile;
        this.password = password;
    }

    public String getMobile() {
        return mobile;
    }

    public void setMobile(String mobile) {
        this.mobile = mobile;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
	}


然后需要一个Client包装类获取网络请求的实例，通常简单的写法是：

    public class RetrofitClient{

		private static Retrofit retrofit;
		private static RetrofitService retrofitService;//私有静态变量，实现单例模式

    	String baseUrl="https://服务器Ip地址/端口号/约定的通用接口地址(如/api/  可以省略)/"//这种敏感信息最好写到gradle里
    
		/*通过get方法获取私有静态变量
		*/
		public static RetrofitService getRetrofitService() {

        if (retrofitService == null) {
            retrofitService = getRetrofit().create(RetrofitService.class);
        }
        return retrofitService;
   		}


	    private static Retrofit getRetrofit() {
        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                    .baseUrl(BuildConfig.URL_BASE)
                    .addConverterFactory(GsonConverterFactory.create())//需导入Gson库
                    .client(getClient())
                    .build();
        }
        return retrofit;
    }
    }

这样一个基本的retrofit网络请求就封装好了
在调用的时候只需要

    RetrofitClient.getRetrofitService().login(loginRequest).enqueue(new Callback<APIResponse<LoginResponse>>() {
                @Override
                public void onResponse(Call<LoginResponse> call, Response<APIResponse<LoginResponse>> response) {
    		//服务器成功收到请求，并返回数据，但并不代表成功，也有可能发生错误，需要根据服务器返回码进行判断，若判断为成功，则可以获取服务器返回的Date并进行响应,如以下代码,APIErrorCode为各种返回码的合集。
           switch (response.body().getCode()) {
                        case APIErrorCode.SUCCESS:
                            loginSuccess(response.body().getData().getUser(), response.body().getData().getToken());//登录成功响应方法
                            break;
                        case APIErrorCode.PasswordError:
                            Toast.makeText(LoginActivity.this, "密码错误", Toast.LENGTH_SHORT).show();
                            break;
                        case APIErrorCode.UserNotExist:
                            Toast.makeText(LoginActivity.this, "用户不存在", Toast.LENGTH_SHORT).show();
                            break;
                    }
                }
    
                @Override
                public void onFailure(Call<APIResponse<LoginResponse>> call, Throwable t) {
    		//服务器没响应，一般由于手机网络断开，或者服务器地址和端口号有误
                }
            });


以上就是retrofit的基本用法


##Rxjava简单介绍
- http://gank.io/post/560e15be2dca930e00da1083 推荐这篇文章，讲的通俗易懂


##Rxjava+Retrofit使用

首先先看看同样实现登录功能Rxjava+Retrofit如何书写。

首先是定义网络请求方法接口类

    
       public interface RetrofitService{
	    /**  
         * 登录
         *
         * @param loginRequest
         * @return
         */
        @POST("user/login")
        Observable<APIResponse<LoginResponse>> login(@Body LoginRequest loginRequest);//这里唯一的区别是将 Call<APIResponse<LoginResponse>>  变为了Rxjava里的可观察对象
    }
	
网络请求的封装

	public class NetworkWrapper {

    private ApiService apiService;
    private Retrofit retrofit;
    String baseUrl="https://服务器Ip地址/端口号/约定的通用接口地址(如/api/  可以省略)/"

    //构造方法私有
    private NetworkWrapper() {

        retrofit = new Retrofit.Builder()
                .client(httpClientBuilder.build())
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(BASE_URL)
                .build();

        apiService = retrofit.create(ApiService.class);
    }

    /**
     * 登录的方法定义
     *
     * @param subscriber 由调用者传过来的观察者对象
     * @param LoginRequest
     */
    public void login(Subscriber<APIResponse<LoginResponse>> subscriber,LoginRequest loginRequest) {
        RetrofitService.login(loginRequest)
                .subscribeOn(Schedulers.io())
                .unsubscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }

    //在访问HttpMethods时创建单例
    private static class SingletonHolder {
        private static final NetworkWrapper INSTANCE = new NetworkWrapper();
    }

    //获取单例
    public static NetworkWrapper get() {
        return SingletonHolder.INSTANCE;
    }
}


	在方法调用时：

  	//新建Subscriber 观察者对象       
    Subscriber<APIResponse<LoginResponse>> subscriber = new Subscriber<APIResponse<LoginResponse>>() {
                    @Override
                    public void onCompleted() {
    					//无论成功，失败，方法结束时的响应函数，一般不做处理
                    }
					     @Override
                    public void onError(Throwable e) {
    					//网络请求失败时的再onError中响应
                        e.printStackTrace();
                    }
    
                    @Override
                    public void onNext(APIResponse<LoginResponse> apiResponse) {
          				//网络请求成功，服务器响应会在 onNext中响应
                    }
    
            };
                NetworkWrapper.get().login(subscriber,LoginRequest);//调用网络封装中的方法


##两种方式对比
由于上面的示例相当简单，两种方法看着并没有什么区别，RXjava+Retrofit看起来要写的代码更多一点.
那为什么需要RXjava+Retrofit来封装网络请求呢？一定有道理

从这个例子看不出来是因为这只是最简单的情况。而一旦情景复杂起来， Callback 形式马上就会开始让人头疼。比如：

假设这么一种情况：你的程序取到的 Date 并不应该直接显示或者直接使用，而是需要先与数据库中的数据进行比对和修正或者合并后再显示。使用 Callback 方式大概可以这么写：
    
	getDate( new Callback<Date>() {
        @Override
        public void success(Date date) {
            new Thread() {
                @Override
                public void run() {//数据库操作应另开线程
                    processUser(date); // 尝试修正 User 数据
                    runOnUiThread(new Runnable() { // 切回 UI 线程
                        @Override
                        public void run() {
                            setDate(date);
                        }
                    });
                }).start();
        }
    
        @Override
        public void failure(RetrofitError error) {
            // Error handling
            ...
        }
    };

虽然解决了，但这种嵌套在更复杂的逻辑中将会更加复杂，最后代码难以阅读
而使用Rxjava 的特性可以写为

    //在网络封装类中便可以统一对返回的数据进行处理，逻辑清晰，调用的时候也不用再写，使用方便
    public void login(Subscriber<APIResponse<LoginResponse>> subscriber,LoginRequest loginRequest) {
            RetrofitService.login(loginRequest)
    				.doOnNext(new Action1<<APIResponse<LoginResponse>>() {
                        @Override
                        public void call(APIResponse<LoginResponse> response) {
    							processDate（response);
                            }
                    })
                    .subscribeOn(Schedulers.io())
                    .unsubscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(subscriber);
        }

再举一个例子：假设 /user 接口并不能直接访问，而需要填入一个在线获取的 token ，代码应该怎么写？
Callback 方式，可以使用嵌套的 Callback：

    @GET("/token")
    public void getToken(Callback<String> callback);
    
    @GET("/user")
    public void getUser(@Query("token") String token, @Query("userId") String userId, Callback<User> callback);
    
    ...
    
    getToken(new Callback<String>() {
        @Override
        public void success(String token) {
            getUser(token, userId, new Callback<User>() {
                @Override
                public void success(User user) {
                    userView.setUser(user);
                }
    
                @Override
                public void failure(RetrofitError error) {
                    // Error handling
                    ...
                }
            };
        }
    
        @Override
        public void failure(RetrofitError error) {
            // Error handling
            ...
        }
    });

没有什么性能问题，但是这种重重嵌套在大型项目中会变得特别繁琐，代码难以阅读

而使用Rxjava+retrofit

    @GET("/token")
	public Observable<String> getToken();

	@GET("/user")
	public Observable<User> getUser(@Query("token") String token, @Query("userId") String userId);
	


	////在调用getToken（）方法后 会返回token对象，通过flatMap进行类型转化，其本质是在Observable<String> 被订阅时的onNext中 调用retrofitServicez.getUser（token，userId）方法
	public void getUser(Subscriber<User> subscriber){
		retrofitService.getToken()
		.flatMap(new Func1<String, Observable<User>>() {
        @Override
        public Observable<User> onNext(String token) {
            return retrofitService.getUser(token, userId);
        })
		 .subscribeOn(Schedulers.io())
                .unsubscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }
	}

这样 获取user的时候直接调用 getUser就行，非常方便，逻辑清楚，其实在retrofit中加入Rxjava就是利用Rxjava最大的特性，线性逻辑，避免写各种复杂的回调，并且切换线程方便。
更多的东西需要熟练掌握Rxjava，我只有浅显的了解，就写到这里。