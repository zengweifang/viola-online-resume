---
layout: article
title:  "使用ztree实现异步加载行政区编辑功能"
disqus: true
categories: basic
---

>简介：使用ztree实现异步加载行政区编辑功能

---

> * 第一步：加载行政区省市数据
> * 第二步：调用$scope.getSource.invoke(params)指令方法，并传入第一步获取的数据
> * 第三步：$.fn.zTree.init($("#treeDemo"), setting, zNodes);生成树形结构
> * 第四步：监听callback－zTreeOnClick事件,传出当前节点id 到controller，当controller获取到子集之后，回调_callBack:getChildrenInfo
> * 第五步：根据controller传过来的子集数据，生成子集树结构，并判断是否已选或者半选，并标记
> * 第六步：监听callback－zTreeOnCheck事件，treeObj.getCheckedNodes(true);获取已选节点，并绑定到$scope.data.regionCheckDTOList
> * 第七步：保存时调用getRegionsData方法,组装数据，传入后端

---

```python
Directive:
(function(){
    "use strict";
    angular.module('app.common')
        .directive('ztreeRegionDir', ztreeRegionDir)
    function ztreeRegionDir() {
        return{
            restrict : 'EA',
            scope :{
                data:'=',//省市的数据
                getCityAllChildren:'&',//将id传入控制器，并调用 通过cityId获取区和街道数据的方法
                result:'=',//最终已选择的对象，传入controller
                getSource:'='
            },
            replace : true,
            templateUrl:'app/common/ztree.dialog.html',
            controller : ['$scope','$timeout','CommonService','logger',function($scope,$timeout,CommonService,logger){
                var zTreeObj;
                // zTree 的参数配置，深入使用请参考 API 文档（setting 配置详解）
                function zTreeOnClick(event, treeId, treeNode){
                    var loader = $(event.target).attr("data-loaded");
                    if(!loader && treeNode.id.length==4){
                        $(event.target).attr("data-loaded",true);
                        var treeObj = $.fn.zTree.getZTreeObj(treeId);
                        //传入参数当前节点id，以及回调函数_callBack
                        $scope.getCityAllChildren({id:treeNode.id,_callBack:getChildrenInfo});
                        //传入子节点数据，并追加到当前父节点后面
                        function getChildrenInfo(params){
                            
                            if(params.length!=0){
                                if(treeNode.checked){
                                    angular.forEach(params,function(item){
                                        item.checked = true;
                                    });
                                }
                                treeObj.addNodes(treeNode, params);
                            };
                        };
                    }
                    
                };

                function cancelHalf(treeNode){
                    if (treeNode.checkedEx) return;
                    var zTree = $.fn.zTree.getZTreeObj("treeDemo");
                    treeNode.halfCheck = false;
                    zTree.updateNode(treeNode); 
                };
                function zTreeOnCheck(event, treeId, treeNode){
                    console.log(treeNode)
                    //手动去掉当前节点半选状态
                    cancelHalf(treeNode);  
                    //监听去调已选择的行政区
                    var treeObj = $.fn.zTree.getZTreeObj(treeId);
                    var data = treeObj.getCheckedNodes(true);
                    for (var i = data.length - 1; i >= 0; i--) {
                        var halfCheck = data[i].getCheckStatus();
                        if (halfCheck.half){//为半选则删除
                            data.splice(i,1);
                        };
                    };
                    var temp = [];
                    angular.forEach(data,function(i){
                        var obj={
                            id:i.id,
                            level:getLevel(i.id)
                        }
                        temp.push(obj)
                    });
                    
                    $scope.$apply(function(){
                        $scope.result.value = temp;

                    })
                    
                    /**
                    *   checked当前节点时，将当前节点以及其所有上级节点 和 regionCheckDTOList（原始已选数组）做对比。
                    *   若存在，则从regionCheckDTOList中移除
                    *   checked为false时，检查子节点，并从已选数组中移除
                    */
                    var regionCheckDTOList = $scope.data.regionCheckDTOList;
                    for (var i = regionCheckDTOList.length - 1; i >= 0; i--) {
                        if(treeNode.id.indexOf(regionCheckDTOList[i].id)==0){
                            regionCheckDTOList.splice(i,1);
                        }
                    }
                    
                    if(!treeNode.checked){
                        for (var i = regionCheckDTOList.length - 1; i >= 0; i--) {
                            if(regionCheckDTOList[i].id.indexOf(treeNode.id)===0){
                                regionCheckDTOList.splice(i,1);
                            }
                        };
                    }
                    $scope.data.regionCheckDTOList = regionCheckDTOList;
                };

                function getLevel(id){
                    switch(parseInt(id.length)){
                        case 9:
                            return 4;
                            break;
                        case 6:
                            return 3;
                            break;
                        case 4:
                            return 2;
                            break;
                        case 2:
                            return 1;
                            break;
                    }
                }
                var setting = {
                    check:{
                        enable: true,
                        chkStyle: "checkbox"
                    },
                    data:{
                        simpleData: {
                            enable: true,
                            idKey: "id",
                            pIdKey: "parentId",
                            rootPId: 0
                        }
                    },
                    callback: {
                        onClick: zTreeOnClick,
                        onCheck: zTreeOnCheck,
                        zTreeOnDblClick : zTreeOnClick
                    }
                };
                
                $scope.$watch('getSource',function(value){
                    var obj = value;
                    obj.invoke = function(source){
                        var temp = source;
                        angular.forEach(temp,function(item){
                            if(item.checked == null){
                                item.halfCheck=true;
                            }
                        })
                        var zNodes = temp;
                        $(document).ready(function(){
                            zTreeObj = $.fn.zTree.init($("#treeDemo"), setting, zNodes);
                        }); 
                    }
                });
                
            }]
        }
    }

})();
```

```python
Html:
<ztree-region-dir result="selectedRegionsData" data="regionsData" 
get-city-all-children='getChildrenData(id,_callBack)' 
get-source="getSource">
</ztree-region-dir>
```

```python
Controller:
//初始化行政区，已选行政区对象，保存时用到
$scope.selectedRegionsData={value:[]};
$scope.getSource={};
//获取的行政区省市的数据
$scope.regionsData = {source : [],regionCheckDTOList:’’};

$scope.getChildrenData = function(id,_callback){
	homeManagementService.regionEditByCityId($stateParams.id,id).then(function(res){
            console.log(res)
            _callback(res.data);第四步

        });
    };
//获取行政区数据
function getRegions(id){
    homeManagementService.regionEdit(id).then(function(res){
        console.log(res)
        if(utilsService.interceptionString(res.code) == appConfig.successCode){
            $scope.homePageCertaintyEditInfo.regionCheckDTOList= res.data.regionCheckDTOList;
            $scope.regionsData.source = res.data.regionTreeDTOList;
            $scope.getSource.invoke(res.data.regionTreeDTOList);第二步
            //已选数据传入指令
            $scope.regionsData.regionCheckDTOList = res.data.regionCheckDTOList;
        }else{
            logger.logError(res.data.message);
        }
    },function(){
        logger.logError("网络链接失败！");
    })
};

$scope.typeInfo="首页类目管理选择内容";
homeManagementService.selectById($stateParams.id).then(function(res){
    console.log(res)
    if(res.data.resultCode==200){
        $scope.categorieDetail = res.data.data
        $scope.imgIconUrl=$scope.categorieDetail.iconImgUrl
        $scope.img123={}
        $scope.img123.name=$scope.categorieDetail.iconImgUrl;
    };
    getRegions($stateParams.id); 第一步
});

//组织行政区数据
function getRegionsData(value,regionTreeDTOList){
    if(regionTreeDTOList.length!=0){
        angular.forEach(regionTreeDTOList,function(i){
            var flag = false;
            angular.forEach(value,function(j){
                if(i.id == j.id){
                    flag=true;
                };
            });
            if(!flag){
                value.push(i);
            }
        });
    };
    return value;
};


保存时调用：
var selectedRegions = getRegionsData($scope.selectedRegionsData.value,
$scope.homePageCertaintyEditInfo.regionCheckDTOList);第七步
```
```python
ztree.dialog.html:
<div>
   <ul id="treeDemo" class="ztree"></ul>
</div>
```












