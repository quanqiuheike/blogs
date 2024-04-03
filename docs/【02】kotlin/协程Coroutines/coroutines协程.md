##### 协程常见用法

```kotlin
GlobalScope.launch {
            kotlin.runCatching {
               val res =  withContext(Dispatchers.IO) {
                    NetworkManager.getInstance().getService(ShareService::class.java).share(map)
                }
                Log.d("LUO","------res: ${res.toString()}")
                handleResponse(res)
            }

        }

```



```kotlin
 GlobalScope.launch(Dispatchers.IO) {
                        delay(500)
                        withContext(Dispatchers.Main) {
                            if (mDataBinding.refreshLayout.isRefreshing) {
                                mDataBinding.refreshLayout.finishRefresh()
                            }
                        }
                    }


 GlobalScope.launch(Dispatchers.IO) {
                            delay(100)
                            withContext(Dispatchers.Main) {
                                viewModel.notifyStarClubJoinStatus()
                            }
                        }
```

```kotlin

 val ex = CoroutineExceptionHandler { coroutineContext, throwable ->
            Log.e(TAG,"log : throwable : "+throwable.message)
        }
        GlobalScope.launch(ex) {
            withContext(Dispatchers.IO) {
                val params = HashMap<String, Any>()
                params.put("reportInfo", string)
                mService.reportException(params)
            }
        }
```

```kotlin

fun report() {
        //协程开启多线程
        val ex = CoroutineExceptionHandler { coroutineContext, throwable ->
            Log.e(TAG, "report : throwable : " + throwable.message)
        }
        GlobalScope.launch(ex) {
            withContext(Dispatchers.IO) {
                synchronized(lock) {
                    //同步执行，保证多线程下，不重复上报
                    var playReportDao = starMakuDb.playReportDao()
                    val reports = playReportDao.getAll()
                    if (reports != null && !reports.isEmpty()) {
                        var paramsMap = HashMap<String, Any>()
                        paramsMap.put("reportInfos", reports)
                        runBlocking {
                            val respose = mService.reportPlay(paramsMap)
                            if (respose.isSuccess) {
                                reports.forEach {
                                    playReportDao.delete(it)
                                }
                            }
                        }
                    }
                }
            }
        }
    }
```

```kotlin
fun reportShare(intent: Intent) {
        val map = intent.getSerializableExtra("share_star_maku") as? HashMap<String?, Any?>
        map?.let {
            val ex = CoroutineExceptionHandler { coroutineContext, throwable ->
                Log.e(TAG, "reportShare : throwable : " + throwable.message)
            }
            GlobalScope.launch(ex) {
                withContext(Dispatchers.IO) {
                    mService.operate(it)
                }
            }
        }
    }
```

```kotlin

 private fun initStarMakuData(starCollection: StarCollectionResp?, isLoadMore: Boolean) {
        launch(block = {
            val videoInfoList = starCollection?.pageInfo?.list
            if (!videoInfoList.isNullOrEmpty()) {
                videoInfoList.forEach {
                    runCatching {
                        val shareComponent = async {
                            val params = hashMapOf<String, Long>()
                            params["ssId"] = it.ssId ?: 0L
                            service.starMakuStarSetInfoListRequest(params)
                        }
                        shareComponent.await().data
                    }.onSuccess { videoInfoList ->
                        if (!videoInfoList.isNullOrEmpty()) {
                            it.scVideoInfoList = videoInfoList
                        }
                    }.onFailure {
                    }
                }
            }
            withContext(Dispatchers.Main) {
                if (isLoadMore) {
                    if (!videoInfoList.isNullOrEmpty()) {
                        adapter.addData(videoInfoList)
                        cursor = videoInfoList[videoInfoList.size - 1].ssId ?: 0L
                        requestState.value =
                            ConstantUtils.LoadingPageState.REQUEST_LOAD_MORE_SUCCESS
                    } else {
                        requestState.value =
                            ConstantUtils.LoadingPageState.REQUEST_LOAD_MORE_EMPTY_DATA
                    }
                } else {
                    if (!videoInfoList.isNullOrEmpty()) {
                        adapter.setList(videoInfoList)
                        cursor = videoInfoList[videoInfoList.size - 1].ssId ?: 0L
                        requestState.value = ConstantUtils.LoadingPageState.REQUEST_LOADING_SUCCESS
                        hasData.value = true
                    } else {
                        requestState.value =
                            ConstantUtils.LoadingPageState.REQUEST_LOADING_EMPTY_DATA
                        publicityImg.value = starCollection?.publicityImg
                        hasData.value = false
                    }
                }
            }
        })
    }
```

```kotlin

GlobalScope.launch {
                            withContext(Dispatchers.IO) {
                                kotlin.runCatching {
                                    NetworkManager.getInstance().getService(AnLiMatchService::class.java).apply(invitationCard.invitationId)
                                }.onSuccess {
                                    withContext(Dispatchers.Main) {
                                        if (it.code == 200) {
                                            it.data?.let { result ->
                                                result.button?.let { button ->
                                                    invitationCard.button = button
                                                    binding.actionBtn.text = button.buttonStr
                                                }
                                                invitationCard.invitationType = result.invitationType
                                            }
                                        } else {
                                            LegoToastUtils.show(if (TextUtils.isEmpty(it.msg)) "网络异常, 请重试" else it.msg)
                                        }
                                    }
                                }.onFailure {
                                    if (null != it && !TextUtils.isEmpty(it.localizedMessage))
                                        LegoToastUtils.show(it.localizedMessage)
                                }
                            }
                        }
```

```kotlin
private fun initStarMakudata(collectionInfo: StarMakuCollectionData?){

        launch(block = {
            var scFavoriteslist = collectionInfo?.list
            if(!scFavoriteslist.isNullOrEmpty()){
                scFavoriteslist.forEach {
                    kotlin.runCatching {
                        val shareComponent = async {
                            val params = hashMapOf<String, Long>()
                            params["ssId"] = it.ssId
                            starCollectionService.starMakuStarSetInfoListRequest(params)
                        }
                        shareComponent.await().data
                    }.onSuccess { starInfoList ->
                        if(!starInfoList.isNullOrEmpty()){
                            it.scVideoInfoList = starInfoList
                        }
                    }.onFailure {
//                        it.message?.let { msg -> Log.e("", msg) }
                    }
                }
            }
            withContext(Dispatchers.Main) {
                starManageURL.value = collectionInfo?.starManageURL
                if(!(collectionInfo?.list.isNullOrEmpty())){
                    //返回有数据
                    if (pageNum == 1) {
                        //清空之前数据
                        mainData.value = Pair(FIRST_LOAD_SUCCESS, collectionInfo)
                    } else {
                        mainData.value = Pair(LOADMORE_SUCCESS, collectionInfo)
                    }
                    pageNum++
                } else {
                    if (pageNum == 1) {
                        //清空之前数据
                        mainData.value = Pair(FIRST_LOAD_EMPTY, null)
                    } else {
                        mainData.value = Pair(LOADMORE_NODATA, null)
                    }
                }
            }
        })
    }
```

```kotlin
fun uploadContacts(activity: FragmentActivity, listener: (Boolean) -> Unit) {
        activity.lifecycleScope.launch {
            val result = withContext(Dispatchers.IO) {
                val list = queryPhoneLog()
                val hashMap = mutableMapOf<String, Any>()
                hashMap["contactVos"] = list

                NetworkManager.getInstance().getService(WebViewApiService::class.java)
                    .uploadContactList(hashMap)
            }
            listener.invoke(result.isSuccess)
        }
    }
```

```kotlin
GlobalScope.launch {
            withContext(Dispatchers.IO){
                kotlin.runCatching {
                    NetworkManager.getInstance().getService(SystemConfigService::class.java).getAppApprovedStatus().data
                }.onSuccess {
                    MMKVUtils.putBoolean(AppConfigsContants.APP_APPROVED_STATUS, it?.androidSwitch?:false)
                }


            }
        }
```

```kotlin
private fun syncDaysTask() {
        val token = MMKVUtils.getString(UserConfigsConstant.TOKEN)
        if (!token.isNullOrEmpty()) {
            GlobalScope.launch {
                withContext(Dispatchers.IO) {
                    try {
                        NetworkManager.getInstance().getService(SystemConfigService::class.java).syncSystemConfig(token)
                    } catch (e: Exception) {

                    }
                    try {
                        //TIM登录
                        LegoLog.i("ccer","延迟登录")
                        TIMHelper.instances.loadUserSig()
                    } catch (e: java.lang.Exception) {

                    }
                }
            }
        }
    }
```



```kotlin
fun loginTIM(userId: String) {
        GlobalScope.launch {
            try {
                withContext(Dispatchers.IO) {
                    TIMManager.getInstance().login(
                            userId,
                            TIMLoginSigUtil.getSig(),
                            onSuccess = {
                                log("TIM登录成功")
                                loginSuccess()
                            },
                            onError = { code, desc ->
                                log("TIM登录失败-本地存储的sig")
                                loginRetry()
                            })
                }

            } catch (e: Exception) {

            }
        }
    }
```

```kotlin
fun authorizeBindFind(source:Int, authCall: AuthCall) {
        if (mReferece.get() == null) {
            return
        }
        val activity = mReferece.get()
        activity?.lifecycleScope?.launch {
            try {
                val bind = withContext(Dispatchers.IO) {
                    mService.authorizeBindFind(source)
                }
                val data = handleResponse(bind)
                if (data.bindResult) {
                    authCall.onSuccess()
                } else {
                    if (data.buttonResult != null) {
                        showBind(data.buttonResult,  authCall)
                    } else {
                        getAuthParams(authCall)
                    }
                }
            }catch (e: Exception) {
                authCall.onFail()
            }


        }
    }
```

```kotlin
GlobalScope.launch {
            withContext(Dispatchers.IO) {
                kotlin.runCatching {
                    NetworkManager.getInstance().getService(DetailService::class.java).sendComment(map)
                }.onSuccess {
                    withContext(Dispatchers.Main) {

                    }
                }.onFailure {
                    if (null != it && !TextUtils.isEmpty(it.localizedMessage))
                        LegoToastUtils.show(it.localizedMessage)
                }
            }
        }
```

```kotlin
GlobalScope.launch {
            withContext(Dispatchers.IO) {
                kotlin.runCatching {
                    NetworkManager.getInstance().getService(DetailService::class.java).contentLike(map)
                }.onSuccess {
                    withContext(Dispatchers.Main) {
                        if (isLike) {
                            setData(likeNumber - 1, false)
                        } else {
                            setData(likeNumber + 1, true)
                        }
                    }
                }.onFailure {
                    if (null != it && !TextUtils.isEmpty(it.localizedMessage))
                        LegoToastUtils.show(it.localizedMessage)
                }
            }
        }
```

```kotlin
launch(block = {
            starSetInfo?.pageInfo?.list?.let{
                it.forEach {
                    kotlin.runCatching {
                        val shareComponent = async {
                            val params = hashMapOf<String, Long>()
                            params["ssId"] = it.ssId?:0L
                            services.starMakuStarSetInfoListRequest(params)
                        }
                        shareComponent.await().data
                    }.onSuccess { starInfoList ->
                        if(!starInfoList.isNullOrEmpty()){
                            it.scVideoInfoList = starInfoList
                        }
                    }.onFailure {
//                        it.message?.let { msg -> Log.e("", msg) }
                    }
                }
            }
            withContext(Dispatchers.Main) {
                updateStarsData(isRefresh, starSetInfo?.pageInfo?.list)
            }
        })
```

```kotlin
GlobalScope.launch {
            try {
                val responseData = withContext(Dispatchers.IO) {
                    NetworkManager.getInstance().getService(ChatService::class.java).loadRoamMsg(params)
                }
                val realData = handleResponse(responseData)
                if (200 == responseData.code) {
                    onSuccess.invoke(realData)
                } else {
                    onError.invoke(responseData.code, responseData.msg)
                }
            } catch (e: Exception) {
                onError.invoke(-1, e.message)
            }
        }
```

