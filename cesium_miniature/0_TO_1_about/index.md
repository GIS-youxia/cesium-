<!--
  我这么记的意义在哪里呢？？？？
    我想知道 Entity 的内部具体的逻辑，很具体的那种
    看过一遍之后，发现 raiseEvent 事件对应的 addEventlistener 具体在哪里加的有点儿找不到，所以打算从 Viewer 从新开始看

    但是刚才去接水，发现自己这么写好像是没什么意义呢！！！！！
    所以我觉得我应该先思考一下：
      关于字体颜色：
        orange - 类的实例
        green - 类 in Cesium
        skyblue - 关键方法
        pink - 关键参数
      这里面有哪些内容需要区分啊？？
        我根本上是想将这条线串起来，期间，涉及到 Cesium 内部的很多类，还有一些参数，然后还有对应的类的内部会使用到的关键方法，

      关于这些细枝末节，无关紧要的事情先不管，把主线先牵出来
 -->

# 应用案例： polyline

var viewer = new Cesium.Viewer("cesiumContainer");

var redLine = [<font color=orange>viewer.entities</font>](#源码解读-从-viewer-内部开始粗略具体还需参考实际源码).<font color=skyblue>[add](#font-colorgreenentitycollectionfont)</font>({<br/>
&emsp;name: "Red line on terrain",<br/>
&emsp;[polyline](#font-colorred在-entityprototype-上定义和时间相关的响应式属性-------动态根据输入的-polyline-属性-成功生成-polyline-模型的关键font): {<br/>
&emsp;&emsp;positions: Cesium.Cartesian3.fromDegreesArray([-75, 35, -125, 35]),<br/>
&emsp;&emsp;width: 5,<br/>
&emsp;&emsp;material: Cesium.Color.RED,<br/>
&emsp;&emsp;clampToGround: true,<br/>
&emsp;},<br/>
});<br/>
<br/>
<br/>
<br/>

# 源码解读： 从 Viewer 内部开始（粗略，具体还需参考实际源码）

# viewer.entities

entities: {<br/>
&emsp;get: function () {<br/>
&emsp;&emsp;return `this._dataSourceDisplay`.[defaultDataSource](#font-colorgreendatasourcedisplayfont).[entities](#font-colorgreencustomdatasourcefont);<br/>
&emsp;},<br/>
},<br/>
<br/>
<br/>

`this._dataSourceDisplay` = ~~<font color=orange>dataSourceDisplay</font>~~;
<br/>
<br/>
<br/>

var dataSourceCollection = options.dataSources;<br/>
if (!defined(dataSourceCollection)) {<br/>
&emsp;dataSourceCollection = new DataSourceCollection();<br/>
}<br/>

var ~~<font color=orange>dataSourceDisplay</font>~~ = new [<font color=green>DataSourceDisplay</font>](#font-colorgreendatasourcedisplayfont)({<br/>
&emsp;scene: scene,<br/>
&emsp;dataSourceCollection: [dataSourceCollection](dataSourceCollection.md),<br/>
});<br/>
<br/>
<br/>

# **<font color=green>#DataSourceDisplay</font>**

```js
/**
 * Visualizes a collection of {@link DataSource} instances.
 * @alias DataSourceDisplay
 * @constructor
 *
 * @param {Object} options Object with the following properties:
 * @param {Scene} options.scene The scene in which to display the data.
 * @param {DataSourceCollection} options.dataSourceCollection The data sources to display.
 * @param {DataSourceDisplay.VisualizersCallback} [options.visualizersCallback=DataSourceDisplay.defaultVisualizersCallback]
 *        A function which creates an array of visualizers used for visualization.
 *        If undefined, all standard visualizers are used.
 */
```

**<font color=skyblue>defaultDataSource</font>**

defaultDataSource: {<br/>
&emsp;get: function () {<br/>
&emsp;&emsp;return this._defaultDataSource;&emsp;&emsp;<font color=red>//</font> this._defaultDataSource = defaultDataSource;&emsp;**<----**&emsp;var defaultDataSource = new **<font color=green>CustomDataSource()</font>**;<br/>
&emsp;},<br/>
},<br/>
<br/>
<br/>

# **<font color=green>#CustomDataSource</font>**

  ```js
  /**
   * Gets the collection of {@link Entity} instances.
   * @memberof CustomDataSource.prototype
   * @type {EntityCollection}
  */
  ```

entities: {<br/>
&emsp;get: function () {<br/>
&emsp;return this._entityCollection;&emsp;&emsp;<font color=red>//</font> this._entityCollection = new **<font color=green>EntityCollection</font>**(this);<br/>
&emsp;},<br/>
},<br/>



<br/><br/>

# **<font color=green>#EntityCollection</font>**

```js
/**
 * Add an entity to the collection.
 *
 * @param {Entity | Entity.ConstructorOptions} entity The entity to be added.
 * @returns {Entity} The entity that was added.
 * @exception {DeveloperError} An entity with <entity.id> already exists in this collection.
 */
  function EntityCollection(owner) {
    this._owner = owner;
    this._entities = new AssociativeArray();
    this._addedEntities = new AssociativeArray();
    this._removedEntities = new AssociativeArray();
    this._changedEntities = new AssociativeArray();
    this._suspendCount = 0;
    this._collectionChanged = new Event();
    this._id = createGuid();
    this._show = true;
    this._firing = false;
    this._refire = false;
  }
```

EntityCollection.prototype.**<font color=skyblue>add</font>** = function (entity) {<br/>
&emsp;if (!(entity instanceof Entity)) {<br/>
&emsp;&emsp;entity = new [<font color=green>Entity</font>](#font-colorgreenentityfont)(entity);//在这里将传入的 option 转换成 Entity<br/>
&emsp;}<br/>
<br/>
&emsp;var id = [entity.id](####Entity); //如果 id 没传，会自动生成一个 Guid<br/>
&emsp;var entities = this._entities;&emsp;&emsp;// this._entities = new <font color=green>AssociativeArray</font>();<br/>
&emsp;if (entities.contains(id)) {<br/>
&emsp;&emsp;throw new RuntimeError("An entity with id " + id + " already exists in this collection.");<br/>
&emsp;}<br/>
<br/>
&emsp;entity.entityCollection = this;<br/>
&emsp;entities.<font color=skyblue>set</font>(id, entity);<br/>
<br/>
&emsp;if (!this._removedEntities.remove(id)) {<br/>
&emsp;&emsp;this._addedEntities.set(id, entity);<br/>
&emsp;}<br/>
&emsp;entity.[**<font color=pink>definitionChanged</font>**](#font-colorpinkdefinitionchangedfont).addEventListener( &emsp;&emsp;//<br/>
&emsp;&emsp;[<font color=skyblue>EntityCollection.prototype._onEntityDefinitionChanged</font>](#font-colorskyblueentitycollectionprototype_onentitydefinitionchangedfont),<br/>
&emsp;&emsp;this<br/>
&emsp;);<br/>
<br/>
&emsp;[fireChangedEvent](#font-colorskybluefirechangedeventfont)(this);<br/>
&emsp;return entity;<br/>
};<br/>


# **<font color=skyblue>##EntityCollection.prototype._onEntityDefinitionChanged</font>**

```js
EntityCollection.prototype._onEntityDefinitionChanged = function (entity) {
  var id = entity.id;
  if (!this._addedEntities.contains(id)) {
    this._changedEntities.set(id, entity);
  }
  fireChangedEvent(this);
};
```

# **<font color=skyblue>##fireChangedEvent</font>**

```js
function fireChangedEvent(collection) {
  if (collection._firing) { // _firing 变量 直在 此函数 和  EntityCollection 函数中出现了 ，共 4 次，默认： false
    collection._refire = true;// _refire 变量 直在 此函数 和  EntityCollection 函数中出现了 ，共 4 次，默认： false
    return;
  }

  if (collection._suspendCount === 0) {
    var added = collection._addedEntities;
    var removed = collection._removedEntities;
    var changed = collection._changedEntities;
    if (changed.length !== 0 || added.length !== 0 || removed.length !== 0) {
      collection._firing = true;
      do {
        collection._refire = false;
        var addedArray = added.values.slice(0);
        var removedArray = removed.values.slice(0);
        var changedArray = changed.values.slice(0);

        added.removeAll();
        removed.removeAll();
        changed.removeAll();
        collection._collectionChanged.raiseEvent(
          collection,
          addedArray,
          removedArray,
          changedArray
        );
      } while (collection._refire);
      collection._firing = false;
    }
  }
}
```

<br/><br/>

# **<font color=green>#Entity</font>**

function Entity(options) {<br/>
&emsp;options = defaultValue(options, defaultValue.EMPTY_OBJECT);<br/>
<br/>
&emsp;var id = options. id;<br/>
&emsp;if (!defined(id)) {<br/>
&emsp;&emsp;id = createGuid();<br/>
&emsp;}<br/>
<br/>
&emsp;// 声明一些属性<br/>
&emsp;this._id = id;<br/>
&emsp;this._definitionChanged = new Event();<br/>
&emsp;this._name = options. *name*; &emsp;&emsp;<font color=red>//</font> 传入的 name<br/><br/>
&emsp;this._show = defaultValue(options.show, true);<br/>
&emsp;this._parent = undefined;<br/>
&emsp;this._propertyNames = [<br/>
&emsp;&emsp;"billboard","box","corridor","cylinder","description","ellipse", //<br/>
&emsp;&emsp;"ellipsoid","label","model","tileset","orientation","path","plane","point","polygon", //<br/>
&emsp;&emsp;"polyline","polylineVolume","position","properties","rectangle","viewFrom","wall",<br/>
&emsp;]; &emsp;&emsp;<font color=red>//</font> 用于 下方 merge 方法中<br/>
<br/>
&emsp;this._polyline = undefined;<br/>
&emsp;this._polylineSubscription = undefined;<br/>
&emsp;this._position = undefined;<br/>
&emsp;this._positionSubscription = undefined;<br/>
&emsp;this._properties = undefined;<br/>
&emsp;this._propertiesSubscription = undefined;<br/>
&emsp;this._children = [];<br/>
&emsp;...<br/>
<br/>

```js
/**
 * Gets or sets the entity collection that this entity belongs to.
 * @type {EntityCollection}
 */
```

&emsp;this.entityCollection = undefined;<br/>
<br/>
&emsp;this.parent = options.parent;<br/>
&emsp;this.**<font color=skyblue>merge</font>**(options);  //通过 merge 方法 对属性进行初始化，内部结合了 传进来的 options<br/>
}<br/>

# ***<font color=red>在 Entity.prototype 上定义和时间相关的响应式属性 - - - 动态根据输入的 polyline 属性 成功生成 Polyline 模型的关键</font>***

```js
Object.defineProperties(Entity.prototype, {
  // 其他的 定义 并没有书写
  /**
   * Gets or sets the polyline.
   * @memberof Entity.prototype
   * @type {PolylineGraphics|undefined}
   */
  polyline: createPropertyTypeDescriptor("polyline", PolylineGraphics),
  /**
   * Gets or sets the polyline volume.
   * @memberof Entity.prototype
   * @type {PolylineVolumeGraphics|undefined}
   */
  properties: createPropertyTypeDescriptor("properties", PropertyBag),
  /**
   * Gets or sets the position.
   * @memberof Entity.prototype
   * @type {PositionProperty|undefined}
   */
  position: createPositionPropertyDescriptor("position"),
});
```

<br/>

# [**<font color=skyblue>##createPropertyTypeDescriptor</font>**](createPropertyTypeDescriptor.md)


# **<font color=skyblue>##merge</font>**

在这个方法里完成 当前 Entity 实例的属性的初始化

```js
/**
 * Assigns each unassigned property on this object to the value
 * of the same property on the provided source object.
 *
 * @param {Entity} source The object to be merged into this object.
 */

Entity.prototype.merge = function (source) {
  //>>includeStart('debug', pragmas.debug);
  if (!defined(source)) {
    throw new DeveloperError("source is required.");
  }
  //>>includeEnd('debug');

  //Name, show, and availability are not Property objects and are currently handled differently.
  //source.show is intentionally ignored because this.show always has a value.
  this.name = defaultValue(this.name, source.name);
  this.availability = defaultValue(this.availability, source.availability);

  var propertyNames = this._propertyNames;
  var sourcePropertyNames = defined(source._propertyNames) ? source._propertyNames : Object.keys(source);
  var propertyNamesLength = sourcePropertyNames.length;

  for (var i = 0; i < propertyNamesLength; i++) {
    var name = sourcePropertyNames[i];

    //While source is required by the API to be an Entity, we internally call this method from the
    //constructor with an options object to configure initial custom properties.
    //So we need to ignore reserved-non-property.
    if (name === "parent" || name === "name" || name === "availability") {
      continue;
    }

    var targetProperty = this[name];
    var sourceProperty = source[name];

    //Custom properties that are registered on the source entity must also
    //get registered on this entity.
    if (!defined(targetProperty) && propertyNames.indexOf(name) === -1) {
      this.addProperty(name);
    }

    if (defined(sourceProperty)) {
      if (defined(targetProperty)) {
        if (defined(targetProperty.merge)) {
          targetProperty.merge(sourceProperty);
        }
      } else if (defined(sourceProperty.merge) && defined(sourceProperty.clone)) {
        this[name] = sourceProperty.clone();
      } else {
        this[name] = sourceProperty;
      }
    }
  }
};
```



# **<font color=pink>##definitionChanged</font>**

```js
/**
   * Gets the event that is raised whenever a property or sub-property is changed or modified.
   * @memberof Entity.prototype
   *
   * @type {Event}
   * @readonly
   */
  definitionChanged: {
    get: function () {
      return this._definitionChanged;
    },
  },
```

<br/>
<br/>

# **<font color=green>#AssociativeArray</font>**

```js
/**
 * A collection of key-value pairs that is stored as a hash for easy
 * lookup but also provides an array for fast iteration.
 * @alias AssociativeArray
 * @constructor
 */
function AssociativeArray() {
  this._array = [];
  this._hash = {};
}

AssociativeArray.prototype.contains = function (key){}
AssociativeArray.prototype.get = function (key){}
AssociativeArray.prototype.set = function (key){}
AssociativeArray.prototype.remove = function (key){}
AssociativeArray.prototype.removeAll = function (key){}
```

**<font color=skyblue>set</font>**

```js
/**
 * Associates the provided key with the provided value.  If the key already
 * exists, it is overwritten with the new value.
 *
 * @param {String|Number} key A unique identifier.
 * @param {*} value The value to associate with the provided key.
 */
AssociativeArray.prototype.set = function (key, value) {
  //>>includeStart('debug', pragmas.debug);
  if (typeof key !== "string" && typeof key !== "number") {
    throw new DeveloperError("key is required to be a string or number.");
  }
  //>>includeEnd('debug');

  var oldValue = this._hash[key];
  if (value !== oldValue) {
    this.remove(key);
    this._hash[key] = value;
    this._array.push(value);
  }
};
```

# **<font color=green>#Event</font>**

```js
/**
 * Registers a callback function to be executed whenever the event is raised.
 * An optional scope can be provided to serve as the <code>this</code> pointer
 * in which the function will execute.
 *
 * @param {Function} listener The function to be executed when the event is raised.
 * @param {Object} [scope] An optional object scope to serve as the <code>this</code>
 *        pointer in which the listener function will execute.
 * @returns {Event.RemoveCallback} A function that will remove this event listener when invoked.
 *
 * @see Event#raiseEvent
 * @see Event#removeEventListener
 */
Event.prototype.addEventListener = function (listener, scope) {
  //>>includeStart('debug', pragmas.debug);
  Check.typeOf.func("listener", listener);
  //>>includeEnd('debug');

  this._listeners.push(listener);
  this._scopes.push(scope);

  var event = this;
  return function () {
    event.removeEventListener(listener, scope);
  };
};
```
