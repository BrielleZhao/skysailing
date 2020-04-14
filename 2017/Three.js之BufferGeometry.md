```javascript
/**
 * @author alteredq / https://alteredqualia.com/
 */
/*
///BufferGeometry类用来和BufferAttribute配合使用,更多细节可以参考官方的样例https://threejs.org/
/// 这个类是另一种创建几何体对象的方式,它将所有的数据包括顶点位置,法线,面,颜色,uv和其它的自定义属性存在缓冲区,
/// 这样可以减少GPU的负荷,BufferGeometry同样也比Geometry对象复杂,增加了使用的难度,这里的属性都是存放在数组中,
/// 比如顶点位置不是Vector3对象,颜色也不是color对象,而是数组.需要访问这些属性,需要从属性缓冲区中读原始数据.
/// NOTE:根据BufferGeometry类特性,我们在创建一些静态对象,实例化后不经常操作的对象时,选择这个类.
///
///
///Example
            var geometry = new THREE.BufferGeometry(); 
 
            // create a simple square shape. We duplicate the top left and bottom right 
            // vertices because each vertex needs to appear once per triangle. 
 
            var vertexPositions = [ [-1.0, -1.0, 1.0], 
                                    [ 1.0, -1.0, 1.0], 
                                    [ 1.0, 1.0, 1.0], 
                                    [ 1.0, 1.0, 1.0], 
                                    [-1.0, 1.0, 1.0], 
                                    [-1.0, -1.0, 1.0] ]; 
 
            var vertices = new Float32Array( vertexPositions.length * 3 ); 
            // three components per vertex 
            // components of the position vector for each vertex are stored 
            // contiguously in the buffer. 
 
            for ( var i = 0; i < vertexPositions.length; i++ ) { 
                vertices[ i*3 + 0 ] = vertexPositions[i][0]; 
                vertices[ i*3 + 1 ] = vertexPositions[i][1]; 
                vertices[ i*3 + 2 ] = vertexPositions[i][2]; 
            } 
 
            // itemSize = 3 because there are 3 values (components) per vertex 
 
            geometry.addAttribute( 'position', new THREE.BufferAttribute( vertices, 3 ) ); 
            var material = new THREE.MeshBasicMaterial( { color: 0xff0000 } ); 
            var mesh = new THREE.Mesh( geometry, material );
///
///
///
*/
///<summary>BufferGeometry</summary>
///<returns type="BufferGeometry">返回在缓冲区创建的几何体对象</returns>
THREE.BufferGeometry = function () {
 
    this.id = THREE.GeometryIdCount ++; //BufferGeometryy对象id属性.
    this.uuid = THREE.Math.generateUUID();  //调用THREE.Math.generateUUID()方法,BufferGeometry对象uuid(通用唯一识别码)属性,
 
    this.name = '';     //BufferGeometry对象name属性,可以为当前对象定义一个名称,初始化为''
 
    this.attributes = {};   //BufferGeometry对象attributes属性,可以为当前对象定义一个名称,初始化为{}
    this.drawcalls = [];    //WebGL分区块绘制,存放区块的数组
    this.offsets = this.drawcalls; // backwards compatibility 向后兼容
                                    //WebGL分区块绘制,存放区块的数组
 
    this.boundingBox = null;    //立方体界限,根据当前几何体生成的立方体界限 { min: new THREE.Vector3(), max: new THREE.Vector3() }
    this.boundingSphere = null; //球体界限,根据当前几何体生成的球体界限   { radius: float }
 
};
 
/****************************************
****下面是BufferGeometry对象提供的功能函数.
****************************************/
THREE.BufferGeometry.prototype = {
 
    constructor: THREE.BufferGeometry,  //构造器,返回对创建此对象的BufferGeometry函数的引用
 
    /*
    ///addAttribute方法给BufferGeometry对象添加属性信息.
    */
    ///<summary>addAttribute</summary>
    ///<param name="name" type="String">属性名称
    ///<param name="attribute" type="THREE.BufferAttribute">属性对象
    ///<returns type="Object3D">返回添加过属性BufferGeometry对象</returns>
    addAttribute: function ( name, attribute ) {
 
        if ( attribute instanceof THREE.BufferAttribute === false ) {   //如果参数attribute不是THREE.BufferAttribute对象
            //提示用户参数错误.使用正确的对象
            console.warn( 'THREE.BufferGeometry: .addAttribute() now expects ( name, attribute ).' );
 
            this.attributes[ name ] = { array: arguments[ 1 ], itemSize: arguments[ 2 ] };
 
            return;     //直接返回
 
        }
 
        this.attributes[ name ] = attribute;    //返回添加过属性BufferGeometry对象
 
    },
 
    /*
    ///getAttribute方法根据属性名(参数name)返回BufferGeometry对象的属性信息.
    */
    ///<summary>getAttribute</summary>
    ///<param name="name" type="String">属性名称
    ///<returns type="THREE.BufferAttribute">返回BufferGeometry对象的属性</returns>
    getAttribute: function ( name ) {
 
        return this.attributes[ name ]; //返回BufferGeometry对象的属性
 
    },
 
    /*
    ///addDrawCall方法将使用三角面构建的BufferGeometry对象.拆分成多个部分多次调用WebGL绘制,每次调用WebGL绘制将使用当前几何体的顶点数组的子集来配置着色器.
    ///这种做法非常有意义,比如你的几何体对象有65535个以上的顶点,WebGL将一次绘制调用的最大顶点限制为65535个.如果将对象拆分成几部分,就不会出问题了.
    ///被拆分的每部分生成这样一种结构组成的数组:{ start: Integer, count: Integer, index: Integer } 
    ///start重新指定在绘制调用第一个顶点索引，
    ///count指定多少个顶点都包括在内，
    ///index指定一个可选的偏移。
    ///NOTE:使用adddrawcall添加绘制调用，而不是直接修改此数组。
    */
    ///<summary>addDrawCall</summary>
    ///<param name="start" type="Number">start重新指定在绘制调用第一个顶点索引，
    ///<param name="count" type="Number">count指定多少个顶点都包括在内，
    ///<param name="indexOffset" type="Number">index指定一个可选的偏移。
    ///<returns type="Array">返回{ start: Integer, count: Integer, index: Integer } 结构组成的数组</returns>
    addDrawCall: function ( start, count, indexOffset ) {
 
        this.drawcalls.push( {
 
            start: start,   //start重新指定在绘制调用第一个顶点索引，
            count: count,   //count指定多少个顶点都包括在内，
            index: indexOffset !== undefined ? indexOffset : 0 //index指定一个可选的偏移。
 
        } );
 
    },
 
    /*
    ///applyMatrix方法对BufferGeometry对象的vertices顶点应用矩阵变换.达到旋转,缩放,移动的目的.
    */
    ///<summary>applyMatrix</summary>
    ///<param name="matrix" type="Matrix4">仿射矩阵
    ///<returns type="BufferGeometry">返回新的BufferGeometry对象</returns>
    applyMatrix: function ( matrix ) {
 
        var position = this.attributes.position;
 
        if ( position !== undefined ) {
 
            matrix.applyToVector3Array( position.array );
            position.needsUpdate = true;    // 将属性position.needsUpdate设置为true
 
        }
 
        var normal = this.attributes.normal;
 
        if ( normal !== undefined ) {
 
            var normalMatrix = new THREE.Matrix3().getNormalMatrix( matrix );
 
            normalMatrix.applyToVector3Array( normal.array );   //对属性数组中的数值应用变换
            normal.needsUpdate = true;
 
        }
 
    },
 
    /*
    ///fromGeometry方法将Geometry对象转换为BufferGeometry对象.
    ///NOTE:settings参数的取值范围是THREE.NoColors || THREE.FaceColors || THREE.VertexColors
    */
    ///<summary>fromGeometry</summary>
    ///<param name="geometry" type="Geometry">Geometry对象
    ///<param :="" name="settings" three.facecolors="" three.nocolors="" type="{">可选参数,用来设置转换后的对象顶点从哪里继承颜色.
    ///<returns type="BufferGeometry">返回新的BufferGeometry对象</returns>
    fromGeometry: function ( geometry, settings ) {
 
        settings = settings || { 'vertexColors': THREE.NoColors };
 
        var vertices = geometry.vertices;
        var faces = geometry.faces;
        var faceVertexUvs = geometry.faceVertexUvs;
        var vertexColors = settings.vertexColors;
        var hasFaceVertexUv = faceVertexUvs[ 0 ].length > 0;
        var hasFaceVertexNormals = faces[ 0 ].vertexNormals.length == 3;
 
        var positions = new Float32Array( faces.length * 3 * 3 );
        this.addAttribute( 'position', new THREE.BufferAttribute( positions, 3 ) );
 
        var normals = new Float32Array( faces.length * 3 * 3 );
        this.addAttribute( 'normal', new THREE.BufferAttribute( normals, 3 ) );
 
        if ( vertexColors !== THREE.NoColors ) {    //如果没有设置settings参数
 
            var colors = new Float32Array( faces.length * 3 * 3 );  //将Geometry对象的三角面的所有顶点颜色一一转换为BufferGeometry对象的属性格式
            this.addAttribute( 'color', new THREE.BufferAttribute( colors, 3 ) );
 
        }
 
        if ( hasFaceVertexUv === true ) {
 
            var uvs = new Float32Array( faces.length * 3 * 2 );
            this.addAttribute( 'uvs', new THREE.BufferAttribute( uvs, 2 ) );
 
        }
 
        for ( var i = 0, i2 = 0, i3 = 0; i < faces.length; i ++, i2 += 6, i3 += 9 ) {    //遍历Geometry对象的三角面数组
 
            var face = faces[ i ];
 
            var a = vertices[ face.a ];
            var b = vertices[ face.b ];
            var c = vertices[ face.c ];
 
            positions[ i3     ] = a.x;  //将位置信息转换
            positions[ i3 + 1 ] = a.y;
            positions[ i3 + 2 ] = a.z;
 
            positions[ i3 + 3 ] = b.x;
            positions[ i3 + 4 ] = b.y;
            positions[ i3 + 5 ] = b.z;
 
            positions[ i3 + 6 ] = c.x;
            positions[ i3 + 7 ] = c.y;
            positions[ i3 + 8 ] = c.z;
 
            if ( hasFaceVertexNormals === true ) {  //将顶点法线转换
 
                var na = face.vertexNormals[ 0 ];
                var nb = face.vertexNormals[ 1 ];
                var nc = face.vertexNormals[ 2 ];
 
                normals[ i3     ] = na.x;
                normals[ i3 + 1 ] = na.y;
                normals[ i3 + 2 ] = na.z;
 
                normals[ i3 + 3 ] = nb.x;
                normals[ i3 + 4 ] = nb.y;
                normals[ i3 + 5 ] = nb.z;
 
                normals[ i3 + 6 ] = nc.x;
                normals[ i3 + 7 ] = nc.y;
                normals[ i3 + 8 ] = nc.z;
 
            } else {
 
                var n = face.normal;    //将三角面法线转换
 
                normals[ i3     ] = n.x;
                normals[ i3 + 1 ] = n.y;
                normals[ i3 + 2 ] = n.z;
 
                normals[ i3 + 3 ] = n.x;
                normals[ i3 + 4 ] = n.y;
                normals[ i3 + 5 ] = n.z;
 
                normals[ i3 + 6 ] = n.x;
                normals[ i3 + 7 ] = n.y;
                normals[ i3 + 8 ] = n.z;
 
            }
 
            if ( vertexColors === THREE.FaceColors ) {
 
                var fc = face.color;
 
                colors[ i3     ] = fc.r;
                colors[ i3 + 1 ] = fc.g;
                colors[ i3 + 2 ] = fc.b;
 
                colors[ i3 + 3 ] = fc.r;
                colors[ i3 + 4 ] = fc.g;
                colors[ i3 + 5 ] = fc.b;
 
                colors[ i3 + 6 ] = fc.r;
                colors[ i3 + 7 ] = fc.g;
                colors[ i3 + 8 ] = fc.b;
 
            } else if ( vertexColors === THREE.VertexColors ) { //将顶点颜色转换
 
                var vca = face.vertexColors[ 0 ];
                var vcb = face.vertexColors[ 1 ];
                var vcc = face.vertexColors[ 2 ];
 
                colors[ i3     ] = vca.r;
                colors[ i3 + 1 ] = vca.g;
                colors[ i3 + 2 ] = vca.b;
 
                colors[ i3 + 3 ] = vcb.r;
                colors[ i3 + 4 ] = vcb.g;
                colors[ i3 + 5 ] = vcb.b;
 
                colors[ i3 + 6 ] = vcc.r;
                colors[ i3 + 7 ] = vcc.g;
                colors[ i3 + 8 ] = vcc.b;
 
            }
 
            if ( hasFaceVertexUv === true ) {   //将顶点uv转换
 
                var uva = faceVertexUvs[ 0 ][ i ][ 0 ];
                var uvb = faceVertexUvs[ 0 ][ i ][ 1 ];
                var uvc = faceVertexUvs[ 0 ][ i ][ 2 ];
 
                uvs[ i2     ] = uva.x;
                uvs[ i2 + 1 ] = uva.y;
 
                uvs[ i2 + 2 ] = uvb.x;
                uvs[ i2 + 3 ] = uvb.y;
 
                uvs[ i2 + 4 ] = uvc.x;
                uvs[ i2 + 5 ] = uvc.y;
 
            }
 
        }
 
        this.computeBoundingSphere()    //重新计算当前对象的球体界限
 
        return this;    //返回新的BufferGeometry对象.
 
    },
 
    /*
    ///computeBoundingBox方法重新计算当前几何体对象的立方体界限,并更新this.boundingBox属性.
    */
    ///<summary>computeBoundingBox</summary>
    ///<returns type="Number">返回重算立方体界限后的几何体对象</returns>
    computeBoundingBox: function () {
 
        if ( this.boundingBox === null ) {
 
            this.boundingBox = new THREE.Box3();
 
        }
 
        var positions = this.attributes[ 'position' ].array;
 
        if ( positions ) {
 
            var bb = this.boundingBox;
 
            if ( positions.length >= 3 ) {
                bb.min.x = bb.max.x = positions[ 0 ];
                bb.min.y = bb.max.y = positions[ 1 ];
                bb.min.z = bb.max.z = positions[ 2 ];
            }
 
            for ( var i = 3, il = positions.length; i < il; i += 3 ) {   //获得当前位置属性数组中的x,y,z坐标的最大最小值,得到立方体界限
 
                var x = positions[ i ];
                var y = positions[ i + 1 ];
                var z = positions[ i + 2 ];
 
                // bounding box
                // 立方体界限
 
                if ( x < bb.min.x ) {
 
                    bb.min.x = x;
 
                } else if ( x > bb.max.x ) {
 
                    bb.max.x = x;
 
                }
 
                if ( y < bb.min.y ) {
 
                    bb.min.y = y;
 
                } else if ( y > bb.max.y ) {
 
                    bb.max.y = y;
 
                }
 
                if ( z < bb.min.z ) {
 
                    bb.min.z = z;
 
                } else if ( z > bb.max.z ) {
 
                    bb.max.z = z;
 
                }
 
            }
 
        }
 
        if ( positions === undefined || positions.length === 0 ) {
 
            this.boundingBox.min.set( 0, 0, 0 );
            this.boundingBox.max.set( 0, 0, 0 );
 
        }
 
        if ( isNaN( this.boundingBox.min.x ) || isNaN( this.boundingBox.min.y ) || isNaN( this.boundingBox.min.z ) ) {
            //如果position数组没有值,提示用户.
            console.error( 'THREE.BufferGeometry.computeBoundingBox: Computed min/max have NaN values. The position attribute is likely to have NaN values.' );
 
        }
 
    },
 
    /*
    ///computeBoundingSphere方法重新计算当前几何体对象的球体界限,并更新this.boundingSphere[]属性.
    */
    ///<summary>computeBoundingSphere</summary>
    ///<returns type="Number">返回重算球体界限后的几何体对象</returns>
    computeBoundingSphere: function () {
 
        var box = new THREE.Box3();
        var vector = new THREE.Vector3();
 
        return function () {
 
            if ( this.boundingSphere === null ) {
 
                this.boundingSphere = new THREE.Sphere();
 
            }
 
            var positions = this.attributes[ 'position' ].array;
 
            if ( positions ) {
 
                box.makeEmpty();
 
                var center = this.boundingSphere.center;
 
                for ( var i = 0, il = positions.length; i < il; i += 3 ) {
 
                    vector.set( positions[ i ], positions[ i + 1 ], positions[ i + 2 ] );
                    box.expandByPoint( vector );
 
                }
 
                box.center( center );
 
                // hoping to find a boundingSphere with a radius smaller than the
                // boundingSphere of the boundingBox:  sqrt(3) smaller in the best case
 
                var maxRadiusSq = 0;
 
                for ( var i = 0, il = positions.length; i < il; i += 3 ) {
 
                    vector.set( positions[ i ], positions[ i + 1 ], positions[ i + 2 ] );
                    maxRadiusSq = Math.max( maxRadiusSq, center.distanceToSquared( vector ) );
 
                }
 
                this.boundingSphere.radius = Math.sqrt( maxRadiusSq );  //计算当前几何体对象的球体界限并更新this.boundingSphere[]属性.
 
                if ( isNaN( this.boundingSphere.radius ) ) {
 
                    console.error( 'THREE.BufferGeometry.computeBoundingSphere(): Computed radius is NaN. The position attribute is likely to have NaN values.' );
 
                }
 
            }
 
        }
 
    }(),
 
    computeFaceNormals: function () {
 
        // backwards compatibility
        //方法已删除, 向后兼容
 
    },
 
    /*
    ///computeVertexNormals方法重新计算BufferGeometry对象的三角面对象顶点的法线向量,face数组中每个元素的vertexNormal属性，一个face3型对象有3个，一个face4型对象有4个，
    /// 但是需要注意的是，被多个表面共享的顶点，其法线向量只有一个，同时受到多个表面的影响。比如中心在原点，三组表面都垂直于轴的立方体，
    /// 其第一象限中的顶点，法线向量是(1,1,1)的归一化。虽然看上去不可思议，平面的顶点的法线居然不是垂直于平面的，但这种指定法线的方法
    /// 在利用平面模拟曲面的时候有很好的效果。
    ///
    /// 定义:顶点法线（Vertex Normal）是过顶点的一个矢量，用于在高洛德着色（Gouraud Shading）中的计算光照和纹理效果。在生成曲面时，
    ///      通常令顶点法线和相邻平面的法线保持等角，如图1，这样进行渲染时，会在平面接缝处产生一种平滑过渡的效果。如果是多边形，
    ///      则令顶点法线等于该点所属平面（三角形）的法线，如图2，以便在接缝处产生突出的边缘。
    /// 
    /// 参考:https://en.wikipedia.org/wiki/Vertex_normal
    /// 参考:https://blog.csdn.net/sophistcxf/article/details/9095911
    /// 参考:https://www.cnblogs.com/flying_bat/archive/2007/10/13/923286.html
    */
    ///<summary>computeVertexNormals</summary>
    ///<returns type="BufferGeometry">返回重算法线向量后的几何体对象</returns>
    computeVertexNormals: function () {
 
        if ( this.attributes[ 'position' ] ) {
 
            var i, il;
            var j, jl;
 
            var nVertexElements = this.attributes[ 'position' ].array.length;
 
            if ( this.attributes[ 'normal' ] === undefined ) {
 
                this.attributes[ 'normal' ] = {
 
                    itemSize: 3,
                    array: new Float32Array( nVertexElements )
 
                };
 
            } else {
 
                // reset existing normals to zero
                // 将现有的法线向量重置为0
 
                for ( i = 0, il = this.attributes[ 'normal' ].array.length; i < il; i ++ ) {
 
                    this.attributes[ 'normal' ].array[ i ] = 0;
 
                }
 
            }
 
            var positions = this.attributes[ 'position' ].array;
            var normals = this.attributes[ 'normal' ].array;
 
            var vA, vB, vC, x, y, z,
 
            pA = new THREE.Vector3(),
            pB = new THREE.Vector3(),
            pC = new THREE.Vector3(),
 
            cb = new THREE.Vector3(),
            ab = new THREE.Vector3();
 
            // indexed elements
            // 元素索引
 
            if ( this.attributes[ 'index' ] ) {
 
                var indices = this.attributes[ 'index' ].array;
 
                var offsets = ( this.offsets.length > 0 ? this.offsets : [ { start: 0, count: indices.length, index: 0 } ] );
 
                for ( j = 0, jl = offsets.length; j < jl; ++ j ) {
 
                    var start = offsets[ j ].start;
                    var count = offsets[ j ].count;
                    var index = offsets[ j ].index;
 
                    for ( i = start, il = start + count; i < il; i += 3 ) {
 
                        vA = index + indices[ i ];
                        vB = index + indices[ i + 1 ];
                        vC = index + indices[ i + 2 ];
 
                        x = positions[ vA * 3 ];
                        y = positions[ vA * 3 + 1 ];
                        z = positions[ vA * 3 + 2 ];
                        pA.set( x, y, z );
 
                        x = positions[ vB * 3 ];
                        y = positions[ vB * 3 + 1 ];
                        z = positions[ vB * 3 + 2 ];
                        pB.set( x, y, z );
 
                        x = positions[ vC * 3 ];
                        y = positions[ vC * 3 + 1 ];
                        z = positions[ vC * 3 + 2 ];
                        pC.set( x, y, z );
 
                        cb.subVectors( pC, pB );
                        ab.subVectors( pA, pB );
                        cb.cross( ab );
 
                        normals[ vA * 3     ] += cb.x;
                        normals[ vA * 3 + 1 ] += cb.y;
                        normals[ vA * 3 + 2 ] += cb.z;
 
                        normals[ vB * 3     ] += cb.x;
                        normals[ vB * 3 + 1 ] += cb.y;
                        normals[ vB * 3 + 2 ] += cb.z;
 
                        normals[ vC * 3     ] += cb.x;
                        normals[ vC * 3 + 1 ] += cb.y;
                        normals[ vC * 3 + 2 ] += cb.z;
 
                    }
 
                }
 
            // non-indexed elements (unconnected triangle soup)
            // 没有索引的元素,说明不是三角面对象.
 
            } else {
 
                for ( i = 0, il = positions.length; i < il; i += 9 ) {
 
                    x = positions[ i ];
                    y = positions[ i + 1 ];
                    z = positions[ i + 2 ];
                    pA.set( x, y, z );
 
                    x = positions[ i + 3 ];
                    y = positions[ i + 4 ];
                    z = positions[ i + 5 ];
                    pB.set( x, y, z );
 
                    x = positions[ i + 6 ];
                    y = positions[ i + 7 ];
                    z = positions[ i + 8 ];
                    pC.set( x, y, z );
 
                    cb.subVectors( pC, pB );
                    ab.subVectors( pA, pB );
                    cb.cross( ab );
 
                    normals[ i     ] = cb.x;
                    normals[ i + 1 ] = cb.y;
                    normals[ i + 2 ] = cb.z;
 
                    normals[ i + 3 ] = cb.x;
                    normals[ i + 4 ] = cb.y;
                    normals[ i + 5 ] = cb.z;
 
                    normals[ i + 6 ] = cb.x;
                    normals[ i + 7 ] = cb.y;
                    normals[ i + 8 ] = cb.z;
 
                }
 
            }
 
            this.normalizeNormals();    //获得当前定点的规范化法线向量
 
            this.normalsNeedUpdate = true;  //设置this.normalsNeedUpdate属性为true
 
        }
 
    },
 
    /*
    ///computeTangents方法重新计算BufferGeometry对象三角面对象顶点的切线空间,
    ///TBN切线是一个有方向的单位长度，沿着网格表面指向水平（U）纹理方向。在Three.js中切线被描述为Vector4，包括x,y,z这些组件定义的矢量，
    ///如果需要，及w用来翻转副法线。
    ///
    /// 参考:https://blog.csdn.net/bonchoix/article/details/8619624
    /// 参考:https://azykise.blog.163.com/blog/static/1730802442010431113010224/
    /// 参考:https://blog.csdn.net/meegomeego/article/details/8605463
    /// 参考:https://shiba.hpe.sh.cn/jiaoyanzu/wuli/showArticle.aspx?articleId=474&classId=4
    ///
    /// NOTE:几何体Geometry对象必须有vertex UV属性,0层将被使用.
    */
    ///<summary>computeTangents</summary>
    ///<returns type="Vector3">返回重算切线空间后的几何体对象</returns>
    computeTangents: function () {
 
        // based on https://www.terathon.com/code/tangent.html
        // (per vertex tangents)
 
        if ( this.attributes[ 'index' ] === undefined ||
             this.attributes[ 'position' ] === undefined ||
             this.attributes[ 'normal' ] === undefined ||
             this.attributes[ 'uv' ] === undefined ) {
            //提示用户有index, position, normal or uv属性,才能计算切线空间
            console.warn( 'Missing required attributes (index, position, normal or uv) in BufferGeometry.computeTangents()' );
            return; //退出
 
        }
        //将数组中的元素转换成顶点,法线向量,uv等对象
        var indices = this.attributes[ 'index' ].array;
        var positions = this.attributes[ 'position' ].array;
        var normals = this.attributes[ 'normal' ].array;
        var uvs = this.attributes[ 'uv' ].array;
 
        var nVertices = positions.length / 3;
 
        if ( this.attributes[ 'tangent' ] === undefined ) {
 
            var nTangentElements = 4 * nVertices;
 
            this.attributes[ 'tangent' ] = {
 
                itemSize: 4,
                array: new Float32Array( nTangentElements )
 
            };
 
        }
 
        var tangents = this.attributes[ 'tangent' ].array;
 
        var tan1 = [], tan2 = [];
 
        for ( var k = 0; k < nVertices; k ++ ) {
 
            tan1[ k ] = new THREE.Vector3();
            tan2[ k ] = new THREE.Vector3();
 
        }
 
        var xA, yA, zA,
            xB, yB, zB,
            xC, yC, zC,
 
            uA, vA,
            uB, vB,
            uC, vC,
 
            x1, x2, y1, y2, z1, z2,
            s1, s2, t1, t2, r;
 
        var sdir = new THREE.Vector3(), tdir = new THREE.Vector3();
        /*
        ///handleTriangle方法重新计算三角面对象的切线空间的TB,TBN.
        */
        ///<summary>handleTriangle</summary>
        ///<param name="a" type="Number">三角面角点a的索引
        ///<param name="b" type="Number">三角面角点b的索引
        ///<param name="c" type="Number">三角面角点c的索引
        function handleTriangle( a, b, c ) {
 
            xA = positions[ a * 3 ];
            yA = positions[ a * 3 + 1 ];
            zA = positions[ a * 3 + 2 ];
 
            xB = positions[ b * 3 ];
            yB = positions[ b * 3 + 1 ];
            zB = positions[ b * 3 + 2 ];
 
            xC = positions[ c * 3 ];
            yC = positions[ c * 3 + 1 ];
            zC = positions[ c * 3 + 2 ];
 
            uA = uvs[ a * 2 ];
            vA = uvs[ a * 2 + 1 ];
 
            uB = uvs[ b * 2 ];
            vB = uvs[ b * 2 + 1 ];
 
            uC = uvs[ c * 2 ];
            vC = uvs[ c * 2 + 1 ];
 
            x1 = xB - xA;
            x2 = xC - xA;
 
            y1 = yB - yA;
            y2 = yC - yA;
 
            z1 = zB - zA;
            z2 = zC - zA;
 
            s1 = uB - uA;
            s2 = uC - uA;
 
            t1 = vB - vA;
            t2 = vC - vA;
 
            r = 1.0 / ( s1 * t2 - s2 * t1 );
 
            sdir.set(
                ( t2 * x1 - t1 * x2 ) * r,
                ( t2 * y1 - t1 * y2 ) * r,
                ( t2 * z1 - t1 * z2 ) * r
            );
 
            tdir.set(
                ( s1 * x2 - s2 * x1 ) * r,
                ( s1 * y2 - s2 * y1 ) * r,
                ( s1 * z2 - s2 * z1 ) * r
            );
 
            tan1[ a ].add( sdir );
            tan1[ b ].add( sdir );
            tan1[ c ].add( sdir );
 
            tan2[ a ].add( tdir );
            tan2[ b ].add( tdir );
            tan2[ c ].add( tdir );
 
        }
 
        var i, il;
        var j, jl;
        var iA, iB, iC;
 
        var offsets = this.offsets;
 
        for ( j = 0, jl = offsets.length; j < jl; ++ j ) {
 
            var start = offsets[ j ].start;
            var count = offsets[ j ].count;
            var index = offsets[ j ].index;
 
            for ( i = start, il = start + count; i < il; i += 3 ) {
 
                iA = index + indices[ i ];
                iB = index + indices[ i + 1 ];
                iC = index + indices[ i + 2 ];
 
                handleTriangle( iA, iB, iC );
 
            }
 
        }
 
        var tmp = new THREE.Vector3(), tmp2 = new THREE.Vector3();
        var n = new THREE.Vector3(), n2 = new THREE.Vector3();
        var w, t, test;
        /*
        ///handleVertex方法重新计算几何体对象顶点的切线空间的TB,TBN.
        */
        ///<summary>handleVertex</summary>
        ///<param name="v" type="Number">顶点a的索引
        function handleVertex( v ) {
 
            n.x = normals[ v * 3 ];
            n.y = normals[ v * 3 + 1 ];
            n.z = normals[ v * 3 + 2 ];
 
            n2.copy( n );
 
            t = tan1[ v ];
 
            // Gram-Schmidt orthogonalize
 
            tmp.copy( t );
            tmp.sub( n.multiplyScalar( n.dot( t ) ) ).normalize();
 
            // Calculate handedness
 
            tmp2.crossVectors( n2, t );
            test = tmp2.dot( tan2[ v ] );
            w = ( test < 0.0 ) ? - 1.0 : 1.0;
 
            tangents[ v * 4     ] = tmp.x;
            tangents[ v * 4 + 1 ] = tmp.y;
            tangents[ v * 4 + 2 ] = tmp.z;
            tangents[ v * 4 + 3 ] = w;
 
        }
 
        for ( j = 0, jl = offsets.length; j < jl; ++ j ) {   //遍历顶点数组
 
            var start = offsets[ j ].start;
            var count = offsets[ j ].count;
            var index = offsets[ j ].index;
 
            for ( i = start, il = start + count; i < il; i += 3 ) {
 
                iA = index + indices[ i ];
                iB = index + indices[ i + 1 ];
                iC = index + indices[ i + 2 ];
 
                handleVertex( iA ); //设置顶点的法线向量
                handleVertex( iB );
                handleVertex( iC );
 
            }
 
        }
 
    },
 
    /*
    ///computeOffsets方法将较大的几何体对象的顶点数限制在indexBufferSize所设置的大小,默认为65535.
    */
    ///<summary>computeTangents</summary>
    ///<returns type="Vector3">返回{ start: Integer, count: Integer, index: Integer } 结构组成的数组</returns>
    /*
        computeOffsets
        计算偏移量
        Compute the draw offset for large models by chunking the index buffer into chunks of 65k addressable vertices.
        当遇到比较大的几何体对象,根据对象的索引值将对象拆分成默认尺寸为65535的块,供WebGL绘制图形使用,
        This method will effectively rewrite the index buffer and remap all attributes to match the new indices.
        这个方法非常对拆分模型非常的游泳.
        WARNING: This method will also expand the vertex count to prevent sprawled triangles across draw offsets.
        注意: 这个方法还可以通过更改indexBufferSize的值,防止不完整的三角面,
        indexBufferSize - Defaults to 65535, but allows for larger or smaller chunks.
        indexBufferSize 属性默认大小事65535,但可以更大或更小.
    */
    computeOffsets: function ( indexBufferSize ) {
 
        var size = indexBufferSize;
        if ( indexBufferSize === undefined )
            size = 65535; //WebGL limits type of index buffer values to 16-bit. 
                          //将WebGL绘制界限限定在16位大小
 
        var s = Date.now();
 
        var indices = this.attributes[ 'index' ].array;
        var vertices = this.attributes[ 'position' ].array;
 
        var verticesCount = ( vertices.length / 3 );
        var facesCount = ( indices.length / 3 );
 
        /*
        console.log(Computing buffers in offsets of +size+ -> indices:+indices.length+ vertices:+vertices.length);
        console.log(Faces to process: +(indices.length/3));
        console.log(Reordering +verticesCount+ vertices.);
        */
 
        var sortedIndices = new Uint16Array( indices.length ); //16-bit buffers
        var indexPtr = 0;
        var vertexPtr = 0;
 
        var offsets = [ { start:0, count:0, index:0 } ];
        var offset = offsets[ 0 ];
 
        var duplicatedVertices = 0;
        var newVerticeMaps = 0;
        var faceVertices = new Int32Array( 6 );
        var vertexMap = new Int32Array( vertices.length );
        var revVertexMap = new Int32Array( vertices.length );
        for ( var j = 0; j < vertices.length; j ++ ) { vertexMap[ j ] = - 1; revVertexMap[ j ] = - 1; }
 
        /*
            Traverse every face and reorder vertices in the proper offsets of 65k.
            遍历所有三角面的顶点,将顶点以65k作为界限划分.
            We can have more than 65k entries in the index buffer per offset, but only reference 65k values.
            我们可以讲缓冲区块划分出比65535跟多的顶点数,但是推荐65535
        */
        for ( var findex = 0; findex < facesCount; findex ++ ) {
            newVerticeMaps = 0;
 
            for ( var vo = 0; vo < 3; vo ++ ) {
                var vid = indices[ findex * 3 + vo ];
                if ( vertexMap[ vid ] == - 1 ) {
                    //Unmapped vertice
                    //没有映射的顶点
                    faceVertices[ vo * 2 ] = vid;
                    faceVertices[ vo * 2 + 1 ] = - 1;
                    newVerticeMaps ++;
                } else if ( vertexMap[ vid ] < offset.index ) {
                    //Reused vertices from previous block (duplicate)
                    //复用之前区块的顶点
                    faceVertices[ vo * 2 ] = vid;
                    faceVertices[ vo * 2 + 1 ] = - 1;
                    duplicatedVertices ++;
                } else {
                    //Reused vertice in the current block
                    //在当前的区块儿内复用顶点
                    faceVertices[ vo * 2 ] = vid;
                    faceVertices[ vo * 2 + 1 ] = vertexMap[ vid ];
                }
            }
 
            var faceMax = vertexPtr + newVerticeMaps;
            if ( faceMax > ( offset.index + size ) ) {
                var new_offset = { start:indexPtr, count:0, index:vertexPtr };
                offsets.push( new_offset );
                offset = new_offset;
 
                //Re-evaluate reused vertices in light of new offset.
                //重新评估复用新区块的顶点
                for ( var v = 0; v < 6; v += 2 ) {
                    var new_vid = faceVertices[ v + 1 ];
                    if ( new_vid > - 1 && new_vid < offset.index )
                        faceVertices[ v + 1 ] = - 1;
                }
            }
 
            //Reindex the face.
            //重新索引三角面
            for ( var v = 0; v < 6; v += 2 ) {
                var vid = faceVertices[ v ];
                var new_vid = faceVertices[ v + 1 ];
 
                if ( new_vid === - 1 )
                    new_vid = vertexPtr ++;
 
                vertexMap[ vid ] = new_vid;
                revVertexMap[ new_vid ] = vid;
                sortedIndices[ indexPtr ++ ] = new_vid - offset.index; //XXX overflows at 16bit
                offset.count ++;
            }
        }
 
        /* Move all attribute values to map to the new computed indices , also expand the vertice stack to match our new vertexPtr. */
        //将新计算的索引映射到所有属性值,并且扩展顶点堆栈匹配新的vertexPtr
        this.reorderBuffers( sortedIndices, revVertexMap, vertexPtr );
        this.offsets = offsets; //重新设置this.offsets属性
 
        /*
        var orderTime = Date.now();
        console.log(Reorder time: +(orderTime-s)+ms);
        console.log(Duplicated +duplicatedVertices+ vertices.);
        console.log(Compute Buffers time: +(Date.now()-s)+ms);
        console.log(Draw offsets: +offsets.length);
        */
 
        return offsets; //返回重算的offsets属性.
    },
 
    /*
    ///merge方法将两个几何体对象或者BufferGeometry里面的几何体对象合并,将属性数组合并.
    */
    ///<summary>merge</summary>
    ///<returns type="BufferGeometry">返回合并后的几何体对象</returns> 
    merge: function () {
 
        console.log( 'BufferGeometry.merge(): TODO' );
 
    },
 
    normalizeNormals: function () {
 
        var normals = this.attributes[ 'normal' ].array;
 
        var x, y, z, n;
 
        for ( var i = 0, il = normals.length; i < il; i += 3 ) {
 
            x = normals[ i ];
            y = normals[ i + 1 ];
            z = normals[ i + 2 ];
 
            n = 1.0 / Math.sqrt( x * x + y * y + z * z );
 
            normals[ i     ] *= n;
            normals[ i + 1 ] *= n;
            normals[ i + 2 ] *= n;
 
        }
 
    },
 
    /*
        reoderBuffers:
        Reorder attributes based on a new indexBuffer and indexMap.
        indexBuffer - Uint16Array of the new ordered indices.
        indexMap - Int32Array where the position is the new vertex ID and the value the old vertex ID for each vertex.
        vertexCount - Amount of total vertices considered in this reordering (in case you want to grow the vertice stack).
    */
    /*
    ///reorderBuffers方法根参数indexBuffer和参数indexMap重新排列缓冲区中的BufferGeometry对象的属性数组
    */
    ///<summary>reorderBuffers</summary>
    ///<param name="indexBuffer" type="Uint16Array">新顺序的索引
    ///<param name="indexMap" type="Int32Array">每个顶点id和新位置和值的映射表
    ///<param name="vertexCount" type="Number">顶点a的索引
    ///<returns type="BufferGeometry">返回合并后的几何体对象</returns> 
    reorderBuffers: function ( indexBuffer, indexMap, vertexCount ) {
 
        /* Create a copy of all attributes for reordering. */
        //创建一个所有属性的副本用来重新排序
        var sortedAttributes = {};
        var types = [ Int8Array, Uint8Array, Uint8ClampedArray, Int16Array, Uint16Array, Int32Array, Uint32Array, Float32Array, Float64Array ];
        for ( var attr in this.attributes ) {
            if ( attr == 'index' )
                continue;
            var sourceArray = this.attributes[ attr ].array;
            for ( var i = 0, il = types.length; i < il; i ++ ) {
                var type = types[ i ];
                if ( sourceArray instanceof type ) {
                    sortedAttributes[ attr ] = new type( this.attributes[ attr ].itemSize * vertexCount );
                    break;
                }
            }
        }
 
        /* Move attribute positions based on the new index map */
        // 根据新的索引表移动属性位置.
        for ( var new_vid = 0; new_vid < vertexCount; new_vid ++ ) {
            var vid = indexMap[ new_vid ];
            for ( var attr in this.attributes ) {
                if ( attr == 'index' )
                    continue;
                var attrArray = this.attributes[ attr ].array;
                var attrSize = this.attributes[ attr ].itemSize;
                var sortedAttr = sortedAttributes[ attr ];
                for ( var k = 0; k < attrSize; k ++ )
                    sortedAttr[ new_vid * attrSize + k ] = attrArray[ vid * attrSize + k ];
            }
        }
 
        /* Carry the new sorted buffers locally */
        // 更新存储在本地缓冲区中的属性
        this.attributes[ 'index' ].array = indexBuffer;
        for ( var attr in this.attributes ) {
            if ( attr == 'index' )
                continue;
            this.attributes[ attr ].array = sortedAttributes[ attr ];
            this.attributes[ attr ].numItems = this.attributes[ attr ].itemSize * vertexCount;
        }
    },
 
    /*clone方法
    ///clone方法克隆一个几何体对象,将属性数组分别复制.
    */
    ///<summary>clone</summary>
    ///<returns type="Vector4">返回克隆的几何体对象</returns> 
    clone: function () {
 
        var geometry = new THREE.BufferGeometry();
 
        var types = [ Int8Array, Uint8Array, Uint8ClampedArray, Int16Array, Uint16Array, Int32Array, Uint32Array, Float32Array, Float64Array ];
 
        for ( var attr in this.attributes ) {
 
            var sourceAttr = this.attributes[ attr ];
            var sourceArray = sourceAttr.array;
 
            var attribute = {
 
                itemSize: sourceAttr.itemSize,
                array: null
 
            };
 
            for ( var i = 0, il = types.length; i < il; i ++ ) {
 
                var type = types[ i ];
 
                if ( sourceArray instanceof type ) {
 
                    attribute.array = new type( sourceArray );
                    break;
 
                }
 
            }
 
            geometry.attributes[ attr ] = attribute;
 
        }
 
        for ( var i = 0, il = this.offsets.length; i < il; i ++ ) {
 
            var offset = this.offsets[ i ];
 
            geometry.offsets.push( {
 
                start: offset.start,
                index: offset.index,
                count: offset.count
 
            } );
 
        }
 
        return geometry;    //返回克隆的几何体对象
 
    },
 
    /*dispose方法
    ///dispose方法从内存中删除对象,释放资源.
    ///NOTE: 当删除几何体对象,不要忘记调用这个方法,否则会导致内存泄露.
    */
    ///<summary>dispose</summary>
    dispose: function () {
 
        this.dispatchEvent( { type: 'dispose' } );  //调度事件.
 
    }
 
};
///EventDispatcher方法应用到当前Geometry对象.
THREE.EventDispatcher.prototype.apply( THREE.BufferGeometry.prototype );
