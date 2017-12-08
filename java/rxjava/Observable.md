# Observable

* `Observable fromIterable(Iterable)`

  将一个 `Iterable` 序列为一个发送它们的 `ObservableSource`

  ``` java
  //ObservableFromIterable.FromIterableDisposable
  void run() {
    	do{
        //while 循环，按顺序调用 onNext
        v = ObjectHelper.requireNonNull(it.next(), "The iterator returned a null value");
        actual.onNext(v);
    	}while (hasNext);
  }
  ```

  ​

* `Observable concatMapDelayError(Function,int,boolean)`

  将 `Observable` 发射的每一项都映射到一个 `ObservableSource`（有多少项就有多少个 `ObservableSource`），逐个订阅它们，并**按顺序**发射它们的值，同时延迟发送的任何错误，直到它们全部终止。

  ``` java
  //ObservableConcatMap.SourceObserver 不延迟错误
  void drain(){
    if(d && empty){
      actual.onComplete();
      return;
    }
    if (!empty) {
    	ObservableSource<? extends U> o;
      try {
        //将发射项映射到 ObservableSource
        o = ObjectHelper.requireNonNull(mapper.apply(t), "The mapper returned a null ObservableSource");
      }atch (Throwable ex) {
        //不延迟错误，立即调用 onError
        actual.onError(ex);
      }
      //没有错误，使用内部 Observer 订阅它
      o.subscribe(inner);
    }
  }
  //ObservableConcatMap.SourceObserver.InnerObserver
  public void onNext(U t){
    //将映射的 ObservableSource 的发射项 onNext
    actual.onNext(t);
  }
  public void onError(Throwable t){
    //不延迟错误，立即调用 onError
    actual.onError(t);
  }
  ```

  ``` java
  //ObservableConcatMap.ConcatMapDelayErrorObserver 延迟错误到最后
  void drain(){
    if (d && empty){
      //当发射结束时，存在 error 则调用 onError，否则调用 onComplete
      Throwable ex = error.terminate();
      if (ex != null){
      	actual.onError(ex);
      }else{
        actual.onComplete();
      }
      return;
    }
    
    if (!empty) {
      ObservableSource<? extends R> o;
      try {
        o = ObjectHelper.requireNonNull(mapper.apply(v), "The mapper returned a null ObservableSource");
      }catch (Throwable ex) {
        	//映射过程中的错误不会延迟
      	actual.onError(error.terminate());
      }
      
      //内部 Observer 订阅
      o.subscribe(observer);	
    }
  }

  //ObservableConcatMap.ConcatMapDelayErrorObserver.DelayErrorInnerObserver
  public void onNext(R value) {
    //将映射的 ObservableSource 的发射项 onNext
  	actual.onNext(value);
  }
  public void onError(Throwable e) {
    //ObservableSource 发送错误，不中断
    ConcatMapDelayErrorObserver<?, R> p = parent;
    if (p.error.addThrowable(e)) {
    	
    }
  }
  ```

* `Observable switchIfEmpty(observableSource other)`

  返回源 `ObservableSource` 发射子项的 `Observable`，如果源为空，则返回备选的 `ObservableSource` 发射子项的 `Observable`。

  ``` java
  //ObservableSwitchIfEmpty.SwitchIfEmptyObserver
  @Override
   public void onNext(T t){
     //empty 默认为 true
     if(empty){
       empty = false;
     }
     actual.onNext(t);
   }

  @Override
  public void onComplete(){
    if(empty){
      //如果 source 为空
      //设置 empty 为 false，防止死循环
      empty = false;
      //订阅备选的 ObservableSource
      other.subscribe(this);
    }else{
      actual.onComple();
    }
  }
  ```

  ​

  ​