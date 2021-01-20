# Glide

### 依赖

```
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.11.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

### Glide加载图片的整体流程

```java
Glide.with(Context).load(url).into(View)
```

**1. with** 目的是创建一个RequestManager

```java
@NonNull
public static RequestManager with(@NonNull Context context) {
  return getRetriever(context).get(context);
}
```

with方法有很多种参数，Context、Activity、Fragment、View都可以传入，主要是获取Context

```Java
@NonNull
private static RequestManagerRetriever getRetriever(@Nullable Context context) {
  // Context could be null for other reasons (ie the user passes in null), but in practice it will
  // only occur due to errors with the Fragment lifecycle.
  Preconditions.checkNotNull(
      context,
      "You cannot start a load on a not yet attached View or a Fragment where getActivity() "
          + "returns null (which usually occurs when getActivity() is called before the Fragment "
          + "is attached or after the Fragment is destroyed).");
  return Glide.get(context).getRequestManagerRetriever();
}
```

getRetriever()负责创建RequestManagerRetriever，或者从Activity或Fragment找出RequestManagerRetriever，Retriever检索者

```java
@NonNull
Glide build(@NonNull Context context) {
  ...
  return new Glide(
      context,
      engine,
      memoryCache,
      bitmapPool,
      arrayPool,
      requestManagerRetriever,
      connectivityMonitorFactory,
      logLevel,
      defaultRequestOptionsFactory,
      defaultTransitionOptions,
      defaultRequestListeners,
      isLoggingRequestOriginsEnabled,
      isImageDecoderEnabledForBitmaps);
}
```

Glide是一个单例，在创建的时候生成很多配置对象，比如RequestManagerRetriever 、缓存对象等。

经过一系列配置后会在GlideBuilder.build()中创建返回Glide对象

```java
@NonNull
public RequestManager get(@NonNull Context context) {
  if (context == null) {
    throw new IllegalArgumentException("You cannot start a load on a null Context");
  } else if (Util.isOnMainThread() && !(context instanceof Application)) {
    if (context instanceof FragmentActivity) {
      return get((FragmentActivity) context);
    } else if (context instanceof Activity) {
      return get((Activity) context);
    } else if (context instanceof ContextWrapper
        // Only unwrap a ContextWrapper if the baseContext has a non-null application context.
        // Context#createPackageContext may return a Context without an Application instance,
        // in which case a ContextWrapper may be used to attach one.
        && ((ContextWrapper) context).getBaseContext().getApplicationContext() != null) {
      return get(((ContextWrapper) context).getBaseContext());
    }
  }
  return getApplicationManager(context);
}
```

经过一系列配置后会在RequestManagerRetriever.get()中创建返回RequestManager对象

RequestManager就是负责管理Request 请求，而且会跟着Fragment或Activity的生命周期

get()方法根据context是不同的实例，根据Context的类型是Application和非Application来获取不同的RequestManager

```java
@NonNull
private RequestManager supportFragmentGet(
    @NonNull Context context,
    @NonNull FragmentManager fm,
    @Nullable Fragment parentHint,
    boolean isParentVisible) {
  SupportRequestManagerFragment current =
      getSupportRequestManagerFragment(fm, parentHint, isParentVisible);
  RequestManager requestManager = current.getRequestManager();
  if (requestManager == null) {
    // TODO(b/27524013): Factor out this Glide.get() call.
    Glide glide = Glide.get(context);
    requestManager =
        factory.build(
            glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
    current.setRequestManager(requestManager);
  }
  return requestManager;
}
```

简述一下上面的RequestManager流程，主要目的是获取对应的FragmentManager，创建一个透明的Fragment；

这样 RequestManager自然就有了生命周期了的通知。

**2. load** 目的是创建一个RequestBuilder，并配置各类参数

```java
public RequestBuilder<Drawable> load(@Nullable String string) {
  return asDrawable().load(string);
}

public RequestBuilder<Drawable> asDrawable() {
  return as(Drawable.class);
}

public <ResourceType> RequestBuilder<ResourceType> as(
    @NonNull Class<ResourceType> resourceClass) {
  return new RequestBuilder<>(glide, this, resourceClass, context);
}
```

load方法有很多种参数，Bitmap、Drawable、String、Uri、File、ResourceId、URL、byte[]、Object都可以传入

as()方法就是构造一个RequestBuilder，其中resourceClass表示资源类型，可以是Drawable、Bitmap、还有GifDrawable

**3. into** 复杂无比，包括生命周期管理、缓存策略、裁剪、动画效果等等

```java
@NonNull
public ViewTarget<ImageView, TranscodeType> into(@NonNull ImageView view) {
  Util.assertMainThread();
  Preconditions.checkNotNull(view);
  BaseRequestOptions<?> requestOptions = this;
  if (!requestOptions.isTransformationSet()
      && requestOptions.isTransformationAllowed()
      && view.getScaleType() != null) {
    // Clone in this method so that if we use this RequestBuilder to load into a View and then
    // into a different target, we don't retain the transformation applied based on the previous
    // View's scale type.
    switch (view.getScaleType()) {
      case CENTER_CROP:
        requestOptions = requestOptions.clone().optionalCenterCrop();
        break;
      case CENTER_INSIDE:
        requestOptions = requestOptions.clone().optionalCenterInside();
        break;
      case FIT_CENTER:
      case FIT_START:
      case FIT_END:
        requestOptions = requestOptions.clone().optionalFitCenter();
        break;
      case FIT_XY:
        requestOptions = requestOptions.clone().optionalCenterInside();
        break;
      case CENTER:
      case MATRIX:
      default:
        // Do nothing.
    }
  }
  return into(
      glideContext.buildImageViewTarget(view, transcodeClass),
      /*targetListener=*/ null,
      requestOptions,
      Executors.mainThreadExecutor());
}
```

BaseRequestOptions它保存着所有requestManager的配置，包括缓存策略、裁剪大小、动画效果、优先级等等。在用于创建reqeust的时候，配置reqeust的属性。

into() 根据scale type设定了一个对应的裁剪策略。接着调用了glideContext.buildImageViewTarget(view, transcodeClass)创建一个ViewTarget。最后重载的方法into()

```java
@NonNull
public <X> ViewTarget<ImageView, X> buildImageViewTarget(
    @NonNull ImageView imageView, @NonNull Class<X> transcodeClass) {
  return imageViewTargetFactory.buildTarget(imageView, transcodeClass);
}

public <Z> ViewTarget<ImageView, Z> buildTarget(
    @NonNull ImageView view, @NonNull Class<Z> clazz) {
  if (Bitmap.class.equals(clazz)) {
    return (ViewTarget<ImageView, Z>) new BitmapImageViewTarget(view);
  } else if (Drawable.class.isAssignableFrom(clazz)) {
    return (ViewTarget<ImageView, Z>) new DrawableImageViewTarget(view);
  } else {
    throw new IllegalArgumentException(
        "Unhandled class: " + clazz + ", try .as*(Class).transcode(ResourceTranscoder)");
  }
}
```

ViewTarget，它集成自BaseTarget，相当于是Glide处理完结果的回调，负责处理最后的结果，并且带有生命周期。

![](https://raw.githubusercontent.com/YanhxCN/Knowledge/main/c98d91c05793490ea956feec7b738722.png)

Glide提供了各种各样的Target

```java
/**
 * A factory responsible for producing the correct type of {@link
 * com.bumptech.glide.request.target.Target} for a given {@link android.view.View} subclass.
 */
public class ImageViewTargetFactory {
  @NonNull
  @SuppressWarnings("unchecked")
  public <Z> ViewTarget<ImageView, Z> buildTarget(
      @NonNull ImageView view, @NonNull Class<Z> clazz) {
    if (Bitmap.class.equals(clazz)) {
      return (ViewTarget<ImageView, Z>) new BitmapImageViewTarget(view);
    } else if (Drawable.class.isAssignableFrom(clazz)) {
      return (ViewTarget<ImageView, Z>) new DrawableImageViewTarget(view);
    } else {
      throw new IllegalArgumentException(
          "Unhandled class: " + clazz + ", try .as*(Class).transcode(ResourceTranscoder)");
    }
  }
}
```

ImageViewTargetFactory 负责创建ViewTarget，如果传入Bitmap则创建一个BitmapImageViewTarget，传入Drawable则创建一个DrawableImageViewTarget。这个clazz就是通过asBitmap()或者asDrawable()初始化RequestBuilder决定的

load()使用的是asDrawable()，这里就会创建一个DrawableImageViewTarget

```java
private <Y extends Target<TranscodeType>> Y into(
    @NonNull Y target,
    @Nullable RequestListener<TranscodeType> targetListener,
    BaseRequestOptions<?> options,
    Executor callbackExecutor) {
  Preconditions.checkNotNull(target);
  if (!isModelSet) {
    throw new IllegalArgumentException("You must call #load() before calling #into()");
  }
  Request request = buildRequest(target, targetListener, options, callbackExecutor);
  Request previous = target.getRequest();
  if (request.isEquivalentTo(previous)
      && !isSkipMemoryCacheWithCompletePreviousRequest(options, previous)) {
    // If the request is completed, beginning again will ensure the result is re-delivered,
    // triggering RequestListeners and Targets. If the request is failed, beginning again will
    // restart the request, giving it another chance to complete. If the request is already
    // running, we can let it continue running without interruption.
    if (!Preconditions.checkNotNull(previous).isRunning()) {
      // Use the previous request rather than the new one to allow for optimizations like skipping
      // setting placeholders, tracking and un-tracking Targets, and obtaining View dimensions
      // that are done in the individual Request.
      previous.begin();
    }
    return target;
  }
  requestManager.clear(target);
  target.setRequest(request);
  requestManager.track(target, request);
  return target;
}
```

into的重载方法，也是Glide的核心

buildRequest() 会创建一个Request请求，内部会根据请求是否出错创建两种请求：

1. ErrorRequestCoordinator 错误图片+目标图片联合请求（请求出错时，如有配置，则启动该请求）
2. ThumbnailRequestCoordinator 缩略图+目标图片联合请求（小图请求较快显示。如有配置，在请求开始时启动）

target.getRequest() 获取Target之前的请求，比如一个ImageView之前可能有一个Request正在请求或者执行完成。

```java
@Override
@Nullable
public Request getRequest() {
  Object tag = getTag();
  Request request = null;
  if (tag != null) {
    if (tag instanceof Request) {
      request = (Request) tag;
    } else {
      throw new IllegalArgumentException(
          "You must not call setTag() on a view Glide is targeting");
    }
  }
  return request;
}

@Override
public void setRequest(@Nullable Request request) {
  setTag(request);
}

private void setTag(@Nullable Object tag) {
  isTagUsedAtLeastOnce = true;
  view.setTag(tagId, tag);
}
```

​	ViewTarget会将之前的Request保存在View的Tag属性中

request.isEquivalentTo(previous)用来判断是否是同样的请求

isSkipMemoryCacheWithCompletePreviousRequest(options, previous))表示之前的请求已经完成并且这次的请求不缓存

requestManager.track(target, request) 开始此次真正的请求

```java
synchronized void track(@NonNull Target<?> target, @NonNull Request request) {
  targetTracker.track(target);
  requestTracker.runRequest(request);
}
```

​	TargetTracker内部收录着所有的target，负责在收取到生命周期的时候通知所有的target，这里相当于target注册了成为观察者。

​	TargetTracker内部使用WeakHashMap存储所有的target

```java
private final Set<Target<?>> targets =
      Collections.newSetFromMap(new WeakHashMap<Target<?>, Boolean>());
```

​	RequestTracker管理控制所有的request，负责启动、停止、管理所有的request。

​	RequestTracker内部也使用WeakHashMap存储所有的request

```java
public void runRequest(@NonNull Request request) {
  requests.add(request);
  if (!isPaused) {
    request.begin();
  } else {
    request.clear();
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      Log.v(TAG, "Paused, delaying request");
    }
    pendingRequests.add(request);
  }
}
```

判断下状态，如果是暂停状态就放进pendingRequests等待，否则begin()

```java
// ErrorRequestCoordinator.java
@Override
public void begin() {
  synchronized (requestLock) {
    if (primaryState != RequestState.RUNNING) {
      primaryState = RequestState.RUNNING;
      primary.begin();
    }
  }
}
// SingleRequest.java
@Override
public void begin() {
  synchronized (requestLock) {
    assertNotCallingCallbacks();
    stateVerifier.throwIfRecycled();
    startTime = LogTime.getLogTime();
    if (model == null) {
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        width = overrideWidth;
        height = overrideHeight;
      }
      // Only log at more verbose log levels if the user has set a fallback drawable, because
      // fallback Drawables indicate the user expects null models occasionally.
      int logLevel = getFallbackDrawable() == null ? Log.WARN : Log.DEBUG;
      onLoadFailed(new GlideException("Received null model"), logLevel);
      return;
    }
    if (status == Status.RUNNING) {
      throw new IllegalArgumentException("Cannot restart a running request");
    }
    // If we're restarted after we're complete (usually via something like a notifyDataSetChanged
    // that starts an identical request into the same Target or View), we can simply use the
    // resource and size we retrieved the last time around and skip obtaining a new size, starting
    // a new load etc. This does mean that users who want to restart a load because they expect
    // that the view size has changed will need to explicitly clear the View or Target before
    // starting the new load.
    if (status == Status.COMPLETE) {
      onResourceReady(resource, DataSource.MEMORY_CACHE);
      return;
    }
    // Restarts for requests that are neither complete nor running can be treated as new requests
    // and can run again from the beginning.
    status = Status.WAITING_FOR_SIZE;
    if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
      onSizeReady(overrideWidth, overrideHeight);
    } else {
      target.getSize(this);
    }
    if ((status == Status.RUNNING || status == Status.WAITING_FOR_SIZE)
        && canNotifyStatusChanged()) {
      target.onLoadStarted(getPlaceholderDrawable());
    }
    if (IS_VERBOSE_LOGGABLE) {
      logV("finished run method in " + LogTime.getElapsedMillis(startTime));
    }
  }
}
// ThumbnailRequestCoordinator.java
@Override
public void begin() {
  synchronized (requestLock) {
    isRunningDuringBegin = true;
    try {
      // If the request has completed previously, there's no need to restart both the full and the
      // thumb, we can just restart the full.
      if (fullState != RequestState.SUCCESS && thumbState != RequestState.RUNNING) {
        thumbState = RequestState.RUNNING;
        thumb.begin();
      }
      if (isRunningDuringBegin && fullState != RequestState.RUNNING) {
        fullState = RequestState.RUNNING;
        full.begin();
      }
    } finally {
      isRunningDuringBegin = false;
    }
  }
}
```

onSizeReady

```java
@Override
public void onSizeReady(int width, int height) {
  stateVerifier.throwIfRecycled();
  synchronized (requestLock) {
    if (IS_VERBOSE_LOGGABLE) {
      logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
    if (status != Status.WAITING_FOR_SIZE) {
      return;
    }
    status = Status.RUNNING;
    float sizeMultiplier = requestOptions.getSizeMultiplier();
    this.width = maybeApplySizeMultiplier(width, sizeMultiplier);
    this.height = maybeApplySizeMultiplier(height, sizeMultiplier);
    if (IS_VERBOSE_LOGGABLE) {
      logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
    }
    loadStatus =
        engine.load(
            glideContext,
            model,
            requestOptions.getSignature(),
            this.width,
            this.height,
            requestOptions.getResourceClass(),
            transcodeClass,
            priority,
            requestOptions.getDiskCacheStrategy(),
            requestOptions.getTransformations(),
            requestOptions.isTransformationRequired(),
            requestOptions.isScaleOnlyOrNoTransform(),
            requestOptions.getOptions(),
            requestOptions.isMemoryCacheable(),
            requestOptions.getUseUnlimitedSourceGeneratorsPool(),
            requestOptions.getUseAnimationPool(),
            requestOptions.getOnlyRetrieveFromCache(),
            this,
            callbackExecutor);
    // This is a hack that's only useful for testing right now where loads complete synchronously
    // even though under any executor running on any thread but the main thread, the load would
    // have completed asynchronously.
    if (status != Status.RUNNING) {
      loadStatus = null;
    }
    if (IS_VERBOSE_LOGGABLE) {
      logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
    }
  }
}
```

onSizeReady调用Engine加载

### Engine

```
public class Engine
    implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
  private final Jobs jobs; //负责缓存EngineJob的管理类
  private final EngineKeyFactory keyFactory; //负责创建缓存的key
  private final MemoryCache cache; //内部是LruCache缓存
  private final EngineJobFactory engineJobFactory; //负责创建EngineJob，EngineJob负责处理DecodeJob的加载结果
  private final ResourceRecycler resourceRecycler; //当资源被移除或者释放时，负责回收资源
  private final LazyDiskCacheProvider diskCacheProvider; //内部是DiskLruCache缓存
  private final DecodeJobFactory decodeJobFactory; //负责创建DecodeJob, DecodeJob负责从缓存或原始资源解码转换
  private final ActiveResources activeResources; //内部是弱引用缓存，当前活跃的缓存资源
}
```

**1. load**

```java
public <R> LoadStatus load(
    GlideContext glideContext,
    Object model,
    Key signature,
    int width,
    int height,
    Class<?> resourceClass,
    Class<R> transcodeClass,
    Priority priority,
    DiskCacheStrategy diskCacheStrategy,
    Map<Class<?>, Transformation<?>> transformations,
    boolean isTransformationRequired,
    boolean isScaleOnlyOrNoTransform,
    Options options,
    boolean isMemoryCacheable,
    boolean useUnlimitedSourceExecutorPool,
    boolean useAnimationPool,
    boolean onlyRetrieveFromCache,
    ResourceCallback cb,
    Executor callbackExecutor) {
  long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;
  // 根据图像类型、原始宽高、目标宽高、转换等各种因素创建一个Key
  EngineKey key =
      keyFactory.buildKey(
          model,
          signature,
          width,
          height,
          transformations,
          resourceClass,
          transcodeClass,
          options);
  EngineResource<?> memoryResource;
  synchronized (this) {
    memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);
    if (memoryResource == null) {
      return waitForExistingOrStartNewJob(
          glideContext,
          model,
          signature,
          width,
          height,
          resourceClass,
          transcodeClass,
          priority,
          diskCacheStrategy,
          transformations,
          isTransformationRequired,
          isScaleOnlyOrNoTransform,
          options,
          isMemoryCacheable,
          useUnlimitedSourceExecutorPool,
          useAnimationPool,
          onlyRetrieveFromCache,
          cb,
          callbackExecutor,
          key,
          startTime);
    }
  }
  // Avoid calling back while holding the engine lock, doing so makes it easier for callers to
  // deadlock.
  cb.onResourceReady(memoryResource, DataSource.MEMORY_CACHE);
  return null;
}
```



```java
public synchronized void start(DecodeJob<R> decodeJob) {
  this.decodeJob = decodeJob;
  // 根据是否从缓存加载选定不同的线程池
  GlideExecutor executor =
      decodeJob.willDecodeFromCache() ? diskCacheExecutor : getActiveSourceExecutor();
  // 加入到线程池中，继续到decodeJob.run()
  executor.execute(decodeJob);
}
```



```java
@Override
public void run() {
  // This should be much more fine grained, but since Java's thread pool implementation silently
  // swallows all otherwise fatal exceptions, this will at least make it obvious to developers
  // that something is failing.
  GlideTrace.beginSectionFormat("DecodeJob#run(model=%s)", model);
  // Methods in the try statement can invalidate currentFetcher, so set a local variable here to
  // ensure that the fetcher is cleaned up either way.
  DataFetcher<?> localFetcher = currentFetcher;
  try {
    if (isCancelled) {
      notifyFailed();
      return;
    }
    runWrapped();
  } catch (CallbackException e) {
    // If a callback not controlled by Glide throws an exception, we should avoid the Glide
    // specific debug logic below.
    throw e;
  } catch (Throwable t) {
    // Catch Throwable and not Exception to handle OOMs. Throwables are swallowed by our
    // usage of .submit() in GlideExecutor so we're not silently hiding crashes by doing this. We
    // are however ensuring that our callbacks are always notified when a load fails. Without this
    // notification, uncaught throwables never notify the corresponding callbacks, which can cause
    // loads to silently hang forever, a case that's especially bad for users using Futures on
    // background threads.
    if (Log.isLoggable(TAG, Log.DEBUG)) {
      Log.d(
          TAG,
          "DecodeJob threw unexpectedly" + ", isCancelled: " + isCancelled + ", stage: " + stage,
          t);
    }
    // When we're encoding we've already notified our callback and it isn't safe to do so again.
    if (stage != Stage.ENCODE) {
      throwables.add(t);
      notifyFailed();
    }
    if (!isCancelled) {
      throw t;
    }
    throw t;
  } finally {
    // Keeping track of the fetcher here and calling cleanup is excessively paranoid, we call
    // close in all cases anyway.
    if (localFetcher != null) {
      localFetcher.cleanup();
    }
    GlideTrace.endSection();
  }
}
private void runWrapped() {
  switch (runReason) {
    case INITIALIZE:
      // 首次获取
      stage = getNextStage(Stage.INITIALIZE);
      currentGenerator = getNextGenerator();
      runGenerators();
      break;
    case SWITCH_TO_SOURCE_SERVICE:
      // 从获取Disk资源失败重新获取
      runGenerators();
      break;
    case DECODE_DATA:
      // 获取资源成功
      decodeFromRetrievedData();
      break;
    default:
      throw new IllegalStateException("Unrecognized run reason: " + runReason);
  }
}
private DataFetcherGenerator getNextGenerator() {
  switch (stage) {
    case RESOURCE_CACHE:
      return new ResourceCacheGenerator(decodeHelper, this);
    case DATA_CACHE:
      return new DataCacheGenerator(decodeHelper, this);
    case SOURCE:
      return new SourceGenerator(decodeHelper, this);
    case FINISHED:
      return null;
    default:
      throw new IllegalStateException("Unrecognized stage: " + stage);
  }
}
```



### Glide缓存策略

Glide使用了类似三级缓存策略，分别是弱引用缓存、LruCache缓存、DiskLruCache缓存和网络加载。

> 默认情况下，Glide 会在开始一个新的图片请求之前检查以下多级的缓存：
> 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
> 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
> 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
> 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？
> 前两步检查图片是否在内存中，如果是则直接返回图片。后两步则检查图片是否在磁盘上，以便快速但异步地返回图片。
> 如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，Uri, Url等）。

### Glide复用机制

**1. 缓存和复用机制的区别和作用**

简单来说，缓存是将数据存储起来，下次需要时就不用重新加载数据，直接拿来即用，作用是加快加载速度、避免相同的数据占用空间，降低内存占用；

复用的意思是重新使用，将已经不需要使用的数据空间重新拿来使用，他的作用是避免频繁申请内存，避免OOM，因为在短时间内快速申请释放内存，因为GC不及时，可能在短时间内来不及回收到足够的空间，导致OOM。所以复用的作用是减少内存抖动。

**2. 复用原理**

（1）内存占用

在讲解glide如何复用bitmap之前，先来了解bitmap的内存占用问题。

在Android3.0之前，Bitmap的内存分配分为两部分，一部分是分配在Dalvik的VM堆中，而像素数据的内存是分配在Native堆中。

而到了Android3.0之后，Bitmap的内存则已经全部分配在VM堆上。意味着我们不需要手动释放bitmap内存，gc内存管理机制会帮忙管理。

（2）inMutable

inMutable是glide能够复用的基石，是bitmapFactory提供的一个参数，表示该bitmap是可变的，支持复用的。

Bitmap是android中最容易造成OOM的元凶之一，在Bitmap的解析参数，BitmapFactory.Options中提供了两个属性：inMutable、inBitmap

当你进行Bitmap的复用，需要设置inMutable为true，inBitmap设置想已经存在的Bitmap。

所谓复用的意思，就是将废弃不用准备recyle的bitmap，重新拿过来使用。

在使用inBitmap参数前，每创建一个Bitmap对象都会分配一块内存，而使用了inBitmap后， Bitmap的内存是被重新利用的。

（3）Bitmap复用使用条件

在Android 4.4之前，仅支持相同大小的bitmap，inSampleSize必须为1，而且必须采用jpeg或png格式。

在Android 4.4之后只有一个限制，就是被复用的bitmap尺寸要大于 新的bitmap，简单来说就是大图可以给小图复用。

（4）inMutable、inBitmap使用

```java
BitmapFactory.Options largeOption = new BitmapFactory.Options();
largeOption.inMutable = true; // 设置inMutable
Bitmap largeBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.large, largeOption)

BitmapFactory.Options smallOption = new BitmapFactory.Options();
smallOption.inBitmap = largeBitmap; // 设置inBitmap被复用的Bitmap
Bitmap smallBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.small, smallOption);
```

（5）Lru算法

虽然说，将不再使用的bitmap保存起来能够避免频繁申请内存，减少内存抖动，但是同时也会耗用内存存储这些已经不再使用的bitmap，因此Glide使用LRU算法，类似LruCache，有一个最大空间限制。

### 设计模式

1.单例模式 —— Glide对象

```java
@NonNull
public static Glide get(@NonNull Context context) {
  if (glide == null) {
    GeneratedAppGlideModule annotationGeneratedModule =
        getAnnotationGeneratedGlideModules(context.getApplicationContext());
    synchronized (Glide.class) {
      if (glide == null) {
        checkAndInitializeGlide(context, annotationGeneratedModule);
      }
    }
  }
  return glide;
}
```

2.建造者模式 —— RequestBuilder对象

```
public class RequestBuilder<TranscodeType> extends BaseRequestOptions<RequestBuilder<TranscodeType>>
    implements Cloneable, ModelTypes<RequestBuilder<TranscodeType>> {
      
  public RequestBuilder<TranscodeType> apply(
      @NonNull BaseRequestOptions<?> requestOptions) {}
  
  public RequestBuilder<TranscodeType> transition(
      @NonNull TransitionOptions<?, ? super TranscodeType> transitionOptions) {}
      
  public RequestBuilder<TranscodeType> listener(
      @Nullable RequestListener<TranscodeType> requestListener) {}
      
  public RequestBuilder<TranscodeType> addListener(
      @Nullable RequestListener<TranscodeType> requestListener) {}
}
```

