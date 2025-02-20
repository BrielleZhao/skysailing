# fabric.js的重要对象
Fabric.js源码分享（一）—— 渲染机制

多图层交互的性能

我们现存两套图片编辑器。分别采用基于canvas API渲染的fabric.js，和基于webgl API+wasm搭建。  我对比了这两个编辑器在多图场景下，图上操作的性能表现。

对于大分辨率多图层（超20张）场景， 图上操作对比fabric.js应用和elsa图片编辑器。  其中图片编辑器在频繁操作的渲染场景，采用了合并提交，限制渲染频次，RAF等优化渲染的手段。

fabric.js

elsa-image-editor v2

用户输入延时

30ms 实时性好

>80ms 轻微延时 

丢帧情况 

30% 丢帧严重 ??

5%

图一 图片编辑器应用图上操作性能

图二 Fabric.js 应用图上操作性能

fabric.js的重要对象

FabricObject是重要的基类，其他形状/文字等所有对象继承该类。 FabricObject持有描边/填充/阴影/裁剪（clipPath）等公共属性。所有的子类继承该基类，并持有一个关键的对象cacheCanvas作为canvas缓存。

Canvas的渲染流程和分层渲染

Canvas的实例通过添加删除移动拖拽等操作发起事件触发渲染时，requestRenderAll方法被执行。但该方法调用时如果上一次执行任务还未完成，那么将什么也不做。 如果需要立即渲染，调用renderCanvas()强制渲染, 它会取消并行的requestAnimationFrame渲染任务。 

/**
   * Append a renderAll request to next animation frame.
   * unless one is already in progress, in that case nothing is done
   * a boolean flag will avoid appending more.
   */
  requestRenderAll() {
    if (!this.nextRenderHandle && !this.disposed && !this.destroyed) {
      this.nextRenderHandle = requestAnimFrame(() => this.renderAndReset());
    }
  }

/**
   * Renders the canvas
   */
  renderAll() {
    this.cancelRequestedRender();
    if (this.destroyed) {
      return;
    }
    this.renderCanvas(this.getContext(), this._objects);
  }

fabric.js同时维护两个渲染上屏的canvas。 两个canvas使用同样的尺寸。

当渲染流程触发FabricCanvas对象会调用renderCanvas() 方法，普通形状文字图片等指定在下层的上屏LowerCanvas上绘制，而上层的UpperCanvas,用于绘制画笔画出的内容以及组操作相关内容。

Canvas会记录上一次render之后，被更新，移动或者改变的的object信息，将其放在一个_objectsToRender数组中。每次触发渲染，仅重新计算和绘制变化的对象所在的区域。


  /**
   * Renders both the top canvas and the secondary container canvas.
   */
  renderAll() {
    this.cancelRequestedRender();
    if (this.destroyed) {
      return;
    }
    // 处理上层Canvas的渲染
    if (this.contextTopDirty && !this._groupSelector && !this.isDrawingMode) {
      this.clearContext(this.contextTop);
      this.contextTopDirty = false;
    }
    if (this.hasLostContext) {
      this.renderTopLayer(this.contextTop);
      this.hasLostContext = false;
    }
    !this._objectsToRender &&
      (this._objectsToRender = this._chooseObjectsToRender());
    // 将_objectsToRender中的产生变化的object进行重新绘制
    this.renderCanvas(this.getContext(), this._objectsToRender);
  }


Canvas继承的SelectableCanvas中实现了具体的renderCanvas()绘制方法，是一次渲染的核心流程。


  /**
   * Renders background, objects, overlay and controls.
   * @param {CanvasRenderingContext2D} ctx
   * @param {Array} objects to render
   */
  renderCanvas(ctx: CanvasRenderingContext2D, objects: FabricObject[]) {
    if (this.destroyed) {
      return;
    }

    const v = this.viewportTransform,
      path = this.clipPath;
    this.calcViewportBoundaries();
    this.clearContext(ctx);
    ctx.imageSmoothingEnabled = this.imageSmoothingEnabled;
    // @ts-expect-error node-canvas stuff
    ctx.patternQuality = 'best';
    this.fire('before:render', { ctx });
    // 渲染背景
    this._renderBackground(ctx);

    ctx.save();
    //apply viewport transform once for all rendering process
    ctx.transform(v[0], v[1], v[2], v[3], v[4], v[5]);
    
    // 遍历所有object并将其绘制到主Canvas
    this._renderObjects(ctx, objects);
    
    ctx.restore();
    if (!this.controlsAboveOverlay && !this.skipControlsDrawing) {
      this.drawControls(ctx);
    }
    // 渲染蒙层/裁剪效果
    if (path) {
      path._set('canvas', this);
      // needed to setup a couple of variables
      // todo migrate to the newer one
      path.shouldCache();
      path._transformDone = true;
      (path as TCachedFabricObject).renderCache({ forClipping: true });
      this.drawClipPathOnCanvas(ctx, path as TCachedFabricObject);
    }
    // 渲染覆盖层
    this._renderOverlay(ctx);
    if (this.controlsAboveOverlay && !this.skipControlsDrawing) {
      this.drawControls(ctx);
    }
    this.fire('after:render', { ctx });

    if (this.__cleanupTask) {
      this.__cleanupTask();
      this.__cleanupTask = undefined;
    }
  }


对象级别的缓存

FabricObject的子类共用基类的render()方法，该方法可以指定一个canvas上下文，并将该对象的完整信息绘制其上。


  /**
   * Renders an object on a specified context
   * @param {CanvasRenderingContext2D} ctx Context to render on
   */
  render(ctx: CanvasRenderingContext2D) {
    // do not render if width/height are zeros or object is not visible
    if (this.isNotVisible()) {
      return;
    }
    // 检查可见
    if (
      this.canvas &&
      this.canvas.skipOffscreen &&
      !this.group &&
      !this.isOnScreen()
    ) {
      return;
    }
    // 保存此前的绘制状态
    ctx.save();
    // 设置源（新绘制的内容）与目标（已经存在的内容）组合方式，，默认为source-over 使用源内容覆盖目标内容
    this._setupCompositeOperation(ctx);
    // 计算应用缩放和倾斜变换后的边界框尺寸
    this.drawSelectionBackground(ctx);
    // 应用对象的变换（例如旋转、缩放、平移等）
    this.transform(ctx);
    // 透明度
    this._setOpacity(ctx);
    // 阴影
    this._setShadow(ctx);
    if (this.shouldCache()) {
      // 没有阴影的大部分对象需要自身的缓存
      // 对象自身绘制到缓存离屏Canvas, 注意，如果没有产生DirtyCache的情况，是不需要重新renderCache的
      (this as TCachedFabricObject).renderCache();
      // 将CacheCanvas的内容直接绘制到原Canvas上
      (this as TCachedFabricObject).drawCacheOnCanvas(ctx);
    } else {
      // 不走缓存的情况
      this._removeCacheCanvas();
      this.drawObject(ctx, false, {});
      this.dirty = false;
    }
    ctx.restore();
  }

同时每个子类都实现的自身的渲染函数 _render()，如下 。  它允许指定一个CanvasContext对象，并将自身绘制到该对象关联的Canvas上。 所以基类Object中的render()方法在更新cacheCanvas缓存画布的时候，会直接调用不同子类中的具体render实现渲染的目的。

/**
   * @private
   * @param {CanvasRenderingContext2D} ctx Context to render on
   */
  _render(ctx: CanvasRenderingContext2D) {
    ctx.imageSmoothingEnabled = this.imageSmoothing;
    if (this.isMoving !== true && this.resizeFilter && this._needsResize()) {
      this.applyResizeFilters();
    }
    // 绘制画布边界
    this._stroke(ctx);
    // 绘制对象自身
    this._renderPaintInOrder(ctx);
  }


离屏渲染和增量渲染

fabric.js实现了对象级别的缓存。 从上面的代码可以看出，如果渲染某一对象时，发现其并没有发生绘制属性的变化，那么无需更新缓存，直接从cacheCanvas中复制一份到上屏主canvas的指定区域即可。 离屏canvas的使用，使渲染对象没有属性变化时无需重算相关渲染属性，极大的减少了计算量，而且对于组合复杂的场景，例如嵌套组，当其中一个对象发生变化时，使用离屏缓存，只需要更新一次主canvas，减少了主canvas重绘的次数。

// Group中的drawObjcect
/**
   * Execute the drawing operation for an object on a specified context
   * @param {CanvasRenderingContext2D} ctx Context to render on
   */
  drawObject(
    ctx: CanvasRenderingContext2D,
    forClipping: boolean | undefined,
    context: DrawContext,
  ) {
    // canvas
    this._renderBackground(ctx);
    for (let i = 0; i < this._objects.length; i++) {
      const obj = this._objects[i];
      // TODO: handle rendering edge case somehow
      if (this.canvas?.preserveObjectStacking && obj.group !== this) {
        ctx.save();
        ctx.transform(...invertTransform(this.calcTransformMatrix()));
        obj.render(ctx);
        ctx.restore();
      } else if (obj.group === this) {
        obj.render(ctx);
      }
    }
    this._drawClipPath(ctx, this.clipPath, context);
  }

下面是一个group中其中一个image对象发生变更的渲染流程：

这意味着，如果存在嵌套组的情况，依然只需要变更一次主画布。

脏矩形算法

fabric.js会在每次对象发生变化时，标记对象所覆盖的区域为脏矩形。 使用BoundingBox包围盒来标记对象的所有几何形状和边界。 上一次所在的位置为旧脏矩形，新的位置为新脏矩形。将旧的脏矩形清除， 并在新的位置上重新绘制对象。没有发生变化的区域，将复制上一次的缓存canvas。


  /**
   * Check if cache is dirty
   * @param {Boolean} skipCanvas skip canvas checks because this object is painted
   * on parent canvas.
   */
  isCacheDirty(skipCanvas = false) {
    if (this.isNotVisible()) {
      return false;
    }
    const canvas = this._cacheCanvas;
    const ctx = this._cacheContext;
    if (canvas && ctx && !skipCanvas && this._updateCacheCanvas()) {
      // in this case the context is already cleared.
      return true;
    } else {
      if (this.dirty || (this.clipPath && this.clipPath.absolutePositioned)) {
        if (canvas && ctx && !skipCanvas) {
          ctx.save();
          ctx.setTransform(1, 0, 0, 1, 0, 0);
          // 清除旧的缓存
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          ctx.restore();
        }
        return true;
      }
    }
    return false;
  }
_updateCacheCanvas() {
  // ...
    // 计算中心点相对坐标
   this.cacheTranslationX =
        Math.round(canvas.width / 2 - drawingWidth) + drawingWidth;
  
   this.cacheTranslationY =
        Math.round(canvas.height / 2 - drawingHeight) + drawingHeight;
      context.translate(this.cacheTranslationX, this.cacheTranslationY);
  // ...
}


  /**
   * Paint the cached copy of the object on the target context.
   * @param {CanvasRenderingContext2D} ctx Context to render on
   */
  drawCacheOnCanvas(this: TCachedFabricObject, ctx: CanvasRenderingContext2D) {
    ctx.scale(1 / this.zoomX, 1 / this.zoomY);
    ctx.drawImage(
      this._cacheCanvas,
      -this.cacheTranslationX,
      -this.cacheTranslationY,
    );
  }


问题探讨

为什么fabric.js丢帧率比较高？

可能：帧率高于100FPS 硬件支持有限, 采用了优先响应用户输入的方案。

如何优化？

截流mousemove? 主canvas渲染使用RAF？ 存在什么问题？ 

用户体验 > 指标

参考

http://fabricjs.com/articles/

https://github.com/fabricjs/fabric.js
