# Single



* `Completable flatMapCompletable(Function<? super T, ? extends CompletableSource>)`

  通过 `Function` 应用于 `Single` 发射的子项，来返回一个 `Completable`。

  ``` java
  //FlatMapCompletableObserver
  @Override
  public void onSuccess(T value) {
    CompletableSource cs;
    try {
      //直接调用 function 返回 CompletableSource
    	 cs = ObjectHelper.requireNonNull(mapper.apply(value), "The mapper returned a null CompletableSource");
      } catch (Throwable ex) {
       //function 出现异常，直接调用 onError
    	  Exceptions.throwIfFatal(ex);
        onError(ex);
        return;
    }
    }
  }

  @Override
  public void onError(Throwable e) {
  	actual.onError(e);
  }
  ```

* `Single compose(SingleTransformer<? super T, ? extends R> transformer)`

  通过使用 `SingleTransformer` 来转化 `Single` 。这个方法是在 `Single` 自身上进行操作，而 `lift` 操作符是操作于 `Single` 的 `SingleObservers`。