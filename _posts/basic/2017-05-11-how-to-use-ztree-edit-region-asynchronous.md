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
		            var treeObj = $.fn.zTree.getZTreeObj(treeId);
		            //传入参数当前节点id，以及回调函数_callBack
		            $scope.getCityAllChildren({id:treeNode.id,_callBack:getChildrenInfo}); 第五步
		            //传入子节点数据，并追加到当前父节点后面
		            function getChildrenInfo(params){
		                console.log(params)
		                if(params.length!=0){
		                    angular.forEach(params,function(item){
		                        if(item.checked == null){
		                            item.halfCheck=true;
		                            // item.checked = true;
		                        };
		                    })
		                    treeObj.halfCheck = false;
		                    treeObj.addNodes(treeNode, params);
		                };
		            };
		        };

		        function zTreeOnCheck(event, treeId, treeNode){
		            //监听去调已选择的行政区
		            // var nodeData = tree.jqxTree('getItem',event.args.element);
		            // var data = tree.jqxTree('getCheckedItems');
		            var treeObj = $.fn.zTree.getZTreeObj(treeId);
		            var data = treeObj.getCheckedNodes(true);
		            console.log(data)
		            var temp = [];
		            angular.forEach(data,function(i){
		                var obj={
		                    id:i.id,
		                    level:i.level+1
		                }
		                temp.push(obj)
		            });
		            
		            $scope.result.value = temp;
		            // $scope.data.regionCheckDTOList = nodeData.id;
		            var regionCheckDTOList = $scope.data.regionCheckDTOList;
		            angular.forEach(regionCheckDTOList,function(item,key){
		                if(item.id == treeNode.id){
		                    regionCheckDTOList.splice(key,1);
		                }
		            })
		            $scope.data.regionCheckDTOList = regionCheckDTOList;
		        };
		        var setting = {
		            check:{
		                enable: true,
		                chkStyle: "checkbox",
		                chkboxType: { "Y": "ps", "N": "ps" },
		                // nocheckInherit:true
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
		                onCheck: zTreeOnCheck
		            }
		        };
                
                $scope.$watch('getSource',function(value){
                    var obj = value;
                    obj.invoke = function(source){
                        var temp = source;
                        angular.forEach(temp,function(item){
                            if(item.checked == null){
                                item.halfCheck=true;
                                // item.checked = true;
                            }else{
                                item.halfCheck=false;
                            }
                        })
                        var zNodes = temp;
                        $(document).ready(function(){
                            zTreeObj = $.fn.zTree.init($("#treeDemo"), setting, zNodes);第三步
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
<ztree-region-dir result="selectedRegionsData" data="regionsData" get-city-all-children='getChildrenData(id,_callBack)' get-source="getSource"></ztree-region-dir>
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
var selectedRegions = getRegionsData($scope.selectedRegionsData.value,$scope.homePageCertaintyEditInfo.regionCheckDTOList);第七步
```
```python
ztree.dialog.html:
<div>
   <ul id="treeDemo" class="ztree"></ul>
</div>
```












