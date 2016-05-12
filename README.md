# MyOkhttpsDemo
android 实用的一款网络框架
一.概述

主要包含如下功能：

    一般的get请求
    一般的post请求
    基于Http的文件上传
    文件下载
    加载图片
    支持请求回调，直接返回对象、对象集合
    支持session的保持

可以下载最新的jar okhttp he latest JAR ，添加依赖就可以用了。注意:okhttp内部依赖okio，别忘了同时导入okio,最新的jar地址：okio the latest JAR

或者gradle 添加

compile 'com.squareup.okhttp:okhttp:2.4.0'
compile 'com.squareup.okio:okio:1.5.0'

1.Okhttp发送一个get请求


//使用Okhttp发送一个get请求
//创建okHttpClient对象
OkHttpClient mOkHttpClient = new OkHttpClient();
//创建一个Request
final Request request = new Request.Builder()
        .url("https://github.com/hongyangAndroid")
        .build();
//new call
Call call = mOkHttpClient.newCall(request);
//请求加入调度
call.enqueue(new Callback()
{
    @Override
    public void onFailure(Request request, IOException e)
    {
    }

    @Override
    public void onResponse(final Response response) throws IOException
    {
        String htmlStr =  response.body().string();
        Log.e(TAG,"html = "+htmlStr);
    }
});


以上就是发送一个get请求的步骤，首先构造一个Request对象，参数最起码有个url，当然你可以通过Request.Builder设置更多的参数比如：header、method等。

    然后通过request的对象去构造得到一个Call对象，类似于将你的请求封装成了任务，既然是任务，就会有execute()和cancel()等方法。

    最后，我们希望以异步的方式去执行请求，所以我们调用的是call.enqueue，将call加入调度队列，然后等待任务执行完成，我们在Callback中即可得到结果。

注意：

onResponse回调的参数是response，一般情况下，比如我们希望获得返回的字符串，可以通过response.body().string()获取；如果希望获得返回的二进制字节数组，则调用response.body().bytes()；如果你想拿到返回的inputStream，则调用response.body().byteStream()，看到这，能拿到返回的inputStream，看到这个能意识到一点，这里支持大文件下载，有inputStream我们就可以通过IO的方式写文件。不过也说明一个问题，这个onResponse执行的线程并不是UI线程。的确是的，如果你希望操作控件，还是需要使用handler等

2.Http Post 携带参数

//Okhttp的post 请求
String url = "www.baidu.com";
FormEncodingBuilder builder = new FormEncodingBuilder();
builder.add("username","张鸿洋");

Request request = new Request.Builder()
        .url(url)
        .post(builder.build())
        .build();
mOkHttpClient.newCall(request).enqueue(new Callback(){
    @Override
    public void onFailure(Request request, IOException e) {

    }

    @Override
    public void onResponse(Response response) throws IOException {

    }
});

post的时候，参数是包含在请求体中的；所以我们通过FormEncodingBuilder。添加多个String键值对，然后去构造RequestBody，最后完成我们Request的构造。

3.基于Http的文件上传

使用 MultipartBuilder。当我们需要做类似于表单上传的时候，就可以使用它来构造我们的requestBody

参考如下代码

//post提交文件
File file = new File("readme123.md");
RequestBody fileBody = RequestBody.create(MediaType.parse("application/octet-stream"), file);
RequestBody requestBody = new MultipartBuilder()
        .type(MultipartBuilder.FORM)
        .addPart(Headers.of(
                "Content-Disposition",
                "form-data; name=\"username\""),
                RequestBody.create(null, "张鸿洋"))
        .addPart(Headers.of(
                "Content-Disposition",
                "form-data; name=\"mFile\";filename=\"wjd.mp4\""), fileBody)
        .build();

    Request request = new Request.Builder()
            .url("https://github.com/prsioner/MyfxUtils3Demo")
            .post(requestBody)
            .build();

    Call call = mOkHttpClient.newCall(request);
     call.enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {
                Log.e(TAG,"request failed ="+request);
            }

            @Override
            public void onResponse(Response response) throws IOException {
                Log.e(TAG,"resposebody ="+response.body().string());
            }
        });

对于图片下载，文件下载；这两个一个是通过回调的Response拿到byte[]然后decode成图片；文件下载，就是拿到inputStream做写文件操作，我们这里就不赘述了，参考泡在网上的日子。

4.封装网络请求代码

列举鸿洋对okhttp的封装，下面是通过url 返回一个string 字符串。

public void getHtml(View view)
    {
        String url = "http://sec.mobile.tiancity.com/server/mobilesecurity/version.xml";
        url="http://www.391k.com/api/xapi.ashx/info.json?key=bd_hyrzjjfb4modhj&size=10&page=1";
        OkHttpUtils
                .get()
                .url(url)
//                .addHeader("Accept-Encoding","")
                .build()
                .execute(new MyStringCallback());

    }

通过OkHttpUtils.get()方法获取到GetBuilder() 对象，调用url设置url到builder对象，调用build()方法返回一个带
参数的requset.builde(),最后返回来的是一个requset对象

public RequestCall build()
{
    if (params != null)
    {
        url = appendParams(url, params);
    }

    return new GetRequest(url, tag, params, headers).build();
}

requset类如下

public class RequestCall
{
    private OkHttpRequest okHttpRequest;
    private Request request;
    private Call call;

    private long readTimeOut;
    private long writeTimeOut;
    private long connTimeOut;

    private OkHttpClient clone;

    public RequestCall(OkHttpRequest request)
    {
        this.okHttpRequest = request;
    }

这个requset对象跟上文的

//创建一个Request
final Request request = new Request.Builder()
        .url("https://github.com/hongyangAndroid")
        .build();

是一个对象，只是后者进行了封装，简化了代码。

之后调用Requset.execure()方法把请求加入调度

public void execute(final RequestCall requestCall, Callback callback)
    {
        if (callback == null)
            callback = Callback.CALLBACK_DEFAULT;
        final Callback finalCallback = callback;

        requestCall.getCall().enqueue(new okhttp3.Callback()
        {
            @Override
            public void onFailure(Call call, final IOException e)
            {
                sendFailResultCallback(call, e, finalCallback);
            }
            @Override
            public void onResponse(final Call call, final Response response)
            {
//                if (response.code() >= 400 && response.code() <= 599)
//                {
//                    try
//                    {
//                        sendFailResultCallback(call, new RuntimeException(response.body().string()), finalCallback);
//                    } catch (IOException e)
//                    {
//                        e.printStackTrace();
//                    }
//                    return;
//                }

                try
                {
                    Object o = finalCallback.parseNetworkResponse(response);
                    sendSuccessResultCallback(o, finalCallback);
                } catch (Exception e)
                {
                    sendFailResultCallback(call, e, finalCallback);

                }

            }
        });
    }

大致过程就是这样，鸿洋正是大神，非常清晰的写好了封装后的可供调用的接口，膜拜下！！

实例代码只是调用了

getHtml(mTv);
getImage(mImageView);

这两个封装的方法，是不是很方便呢，其他网络请求类似。


参考：泡在网上的日子：http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html

鸿洋的博客：http://blog.csdn.net/lmj623565791/article/details/47911083
