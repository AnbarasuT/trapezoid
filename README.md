trapezoid
=========

shape



var _this;
window.QuadriAngles = function (canvas, details){
    _this = this;
    this.isTouchDevice = ('ontouchstart' in window) || ('onmsgesturechange' in window);
    
    this.canvas = canvas;
    this.context = this.canvas[0].getContext('2d');
    this.canvas.parent().append('<div id="dragArea"/>');
    $('#dragArea').css({'position':'absolute', 'left':'10px','top':'22px','width':(this.canvas.width()-20)+'px', 'height':(this.canvas.height()-50)+'px'})
    if (this.isTouchDevice) {
        window.addEventListener('touchstart', function (event) { _this.onTouchStart(event);});
        $('#dragArea')[0].addEventListener('touchmove', function (event) { _this.onTouchMove(event);});
        window.addEventListener('touchend', function (event) { _this.onTouchEnd(event)});
    }
    else {
        window.addEventListener('mousedown', function (event) { _this.onTouchStart(event);});
        $('#dragArea')[0].addEventListener('mousemove', function (event) { _this.onTouchMove(event);});
        window.addEventListener('mouseup', function (event) { _this.onTouchEnd(event)});
    }
    
    this.quadLabels={
        nodes: ["A", "B", "C", "D"],
        edges: ["k", "l", "m", "n"],
    }
    
    this.quadStyles={
        lineWidth:2,
        strokeColor:"#000",
        fillColor:"rgba(153, 204, 255, 0.8)",
    }
    this.pointsStyle={
        lineWidth:1,
        strokeColor:"#000",
        fillColor:"rgba(255, 0, 0, 0.6)",
    }
    this.pixelPerUnit = 30;
    this.alignX = this.canvas.width()/2;
    this.alignY = this.canvas.height()/2;
    this.selectedShape = "trapezoid";
    this.shapes = [
                   {
                    name:'trapezoid',
                    points: [{x:this.alignX-150, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX, y:this.alignY}, {x:this.alignX-100, y:this.alignY}]
                   },
                ];
    
    this.shapeTouchPoint = -1;
    
    this.init();
}
QuadriAngles.prototype = {
    init:function(){
        console.log("Activity Initialized...");
        this.initializeDashedLine(this.context);
        this.clearAll();
        
        this.drawShape();
        this.showAngle();
    },
    clearAll:function (){
        this.context.clearRect(0, 0, this.canvas.width(), this.canvas.height());
    },
    reset:function(){
        this.selectedShape = "trapezoid";
        this.shapes = [
                   {
                    name:'trapezoid',
                    points: [{x:this.alignX-150, y:this.alignY-100}, {x:this.alignX+100, y:this.alignY-100}, {x:this.alignX, y:this.alignY}, {x:this.alignX-100, y:this.alignY}]
                   },
                ];
        
        this.init();
    },
    drawShape:function(){
        for (var shape=0; shape<this.shapes.length;shape++) {
            if (this.shapes[shape].name == this.selectedShape) {
                var points = this.shapes[shape].points;
                this.drawQuad(points[0].x, points[0].y, points[1].x, points[1].y, points[2].x, points[2].y, points[3].x, points[3].y, this.quadStyles, this.quadLabels);
                break;
            }
        }
    },
    drawQuad:function(startX, startY, x1, y1, x2, y2, x3, y3, styles, labels){
        this.context.beginPath();
        this.context.lineWidth=styles.lineWidth;
        this.context.strokeStyle=styles.strokeColor;
        this.context.fillStyle=styles.fillColor;
        this.context.moveTo(startX,startY);
        this.context.lineTo(x1,y1);
        this.context.lineTo(x2,y2);
        this.context.lineTo(x3,y3);
        this.context.lineTo(startX,startY);
        this.context.dashStyle=[5,4] ;
        this.context.dashedLine(startX, startY, x2,y2);
        this.context.dashedLine(x1, y1, x3,y3);
        this.context.fill();
        this.context.stroke();
        
        var midX = (startX+x1+x2+x3)/4;
        var midY = (startY+y1+y2+y3)/4;
        
        this.context.beginPath();
        this.context.fillStyle="#000";
        this.context.font="italic 16px Verdana";
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:startX, y:startY});
        dist = this.FindDistanceFromPoints({x:midX, y:midY}, {x:startX, y:startY});
        this.context.fillText(labels.nodes[0], midX+(dist+20) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist+10) * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x1, y:y1});
        dist = this.FindDistanceFromPoints({x:midX, y:midY}, {x:x1, y:y1});
        this.context.fillText(labels.nodes[1], midX+(dist+10) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist+10) * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x2, y:y2});
        dist = this.FindDistanceFromPoints({x:midX, y:midY}, {x:x2, y:y2});
        this.context.fillText(labels.nodes[2], midX+(dist+15) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist+15) * Math.sin(pointAng.angle * (Math.PI / 180)));
        
        pointAng = this.FindAngleFromPoints({x:midX, y:midY}, {x:x3, y:y3});
        dist = this.FindDistanceFromPoints({x:midX, y:midY}, {x:x3, y:y3});
        this.context.fillText(labels.nodes[3], midX+(dist+25) * Math.cos(pointAng.angle * (Math.PI / 180)),midY+(dist+25) * Math.sin(pointAng.angle * (Math.PI / 180)));
        this.context.fill();
        
        var midPoint = this.FindIntersectionPointFrom4Points({x:startX, y:startY}, {x:x2,y:y2},{x:x1,y:y1},{x:x3,y:y3});
        this.context.fillText("E", midPoint.x-5,midPoint.y+20);
        this.context.fill();
        
        this.drawPoint(startX, startY, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x1, y1, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x2, y2, 6, 0, 2, this.pointsStyle);
        this.drawPoint(x3, y3, 6, 0, 2, this.pointsStyle);
        
        
        this.drawPoint(midPoint.x, midPoint.y, 6, 0, 2, this.pointsStyle);
        
    },
    drawLine:function(point1, point2){
        this.context.beginPath();
        this.context.lineWidth=this.lineStyles.lineWidth;
        this.context.strokeStyle=this.lineStyles.strokeColor;
        this.context.moveTo(point1.x,point1.y);
        this.context.lineTo(point2.x,point2.y);
        this.context.stroke();
    },
    drawHandle:function(point1, point2){
        this.context.beginPath();
        this.context.lineWidth=this.lineStyles.lineWidth;
        this.context.strokeStyle=this.lineStyles.strokeColor;
        this.context.moveTo(point1.x,point1.y);
        this.context.lineTo(point2.x,point2.y);
        this.context.stroke();
        this.drawPoint(point1.x, point1.y, 10, 0, 360, this.pointsStyle);
    },
    drawPoint:function(x, y, radius, sAngle, eAngle, styles){
        this.context.beginPath();
        this.context.lineWidth=styles.lineWidth;
        this.context.strokeStyle=styles.strokeColor;
        this.context.fillStyle=styles.fillColor;
        this.context.arc(x,y,radius,sAngle,eAngle*Math.PI);
        this.context.fill();
        this.context.stroke();
    },
    showAngle:function(){
        var points = [];
        for (var shape=0; shape<this.shapes.length;shape++) {
            if (this.shapes[shape].name == this.selectedShape) {
                points = this.shapes[shape].points;
                break;
            }
        }
        var midPoint = this.FindIntersectionPointFrom4Points(points[0], points[2],points[1],points[3]);
        
        var angleA =  this.FindAngleFrom3Points(points[3], points[0], points[1]);
        var angleB =  this.FindAngleFrom3Points(points[0], points[1], points[2]);
        var angleC =  this.FindAngleFrom3Points(points[1], points[2], points[3]);
        var angleD =  this.FindAngleFrom3Points(points[2], points[3], points[0]);
        var angleE =  this.FindAngleFrom3Points(points[0], midPoint, points[1]);
        
        var abLen =  this.FindDistanceFromPoints(points[0], points[1]);
        var bcLen =  this.FindDistanceFromPoints(points[1], points[2]);
        var cdLen =  this.FindDistanceFromPoints(points[2], points[3]);
        var daLen =  this.FindDistanceFromPoints(points[3], points[0]);
        var acLen =  this.FindDistanceFromPoints(points[0], points[2]);
        var bdLen =  this.FindDistanceFromPoints(points[1], points[3]);
        
        abLen = (abLen/this.pixelPerUnit).toFixed(1);
        bcLen = (bcLen/this.pixelPerUnit).toFixed(1);
        cdLen = (cdLen/this.pixelPerUnit).toFixed(1);
        daLen = (daLen/this.pixelPerUnit).toFixed(1);
        acLen = (acLen/this.pixelPerUnit).toFixed(1);
        bdLen = (bdLen/this.pixelPerUnit).toFixed(1);
        
        $('.aangle').text(angleA.toFixed(1));
        $('.bangle').text(angleB.toFixed(1));
        $('.cangle').text(angleC.toFixed(1));
        $('.dangle').text(angleD.toFixed(1));
        $('.eangle').text(angleE.toFixed(1));

        $('.ablen').text(abLen);
        $('.bclen').text(bcLen);
        $('.cdlen').text(cdLen);
        $('.dalen').text(daLen);
        $('.aclen').text(acLen);
        $('.bdlen').text(bdLen);
        
        this.displaySymbols(points);
    },
    displaySymbols:function(points){
        this.context.beginPath();
        this.context.lineWidth="1";
        this.context.strokeStyle="#000";
        var midPoint = this.FindIntersectionPointFrom4Points(points[0], points[2],points[1],points[3]);
        var ang1 = this.FindAngleFromPoints(points[0], midPoint);
        var ang2 = this.FindAngleFromPoints(points[1], midPoint);
        ang1.angle +=180;
        ang2.angle +=180;
        var ang3 = ang1.angle - ang2.angle;
        this.context.arc(midPoint.x, midPoint.y, 20, ang1.angle * (Math.PI / 180), ang2.angle * (Math.PI / 180));
        this.context.stroke();
    },
    onTouchStart:function(event){
        if (!event)event=window.event;
        var moX, moY;
        
        moX = _this.touchCoordsFromEvent(event, true)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, true)[0].y - _this.canvas.offset().top;
        
        var points = [];
        for (var shape=0; shape<_this.shapes.length;shape++) {
            if (_this.shapes[shape].name == _this.selectedShape) {
                points = _this.shapes[shape].points;
                break;
            }
        }
        
        var midX = midY = 0;
        for(var point =0; point < points.length; point++)
        {
            midX += points[point].x;
            midY += points[point].y;
        }
        midX = midX / points.length;
        midY = midY / points.length;
        this.pointsAng = [];
        this.pointsDist = [];
        this.shapeMidX = midX;
        this.shapeMidY = midY;
        for(var point =0; point < points.length; point++)
        {
            var ang = _this.FindAngleFromPoints({x:midX, y:midY}, {x:points[point].x, y:points[point].y});
            var dist = _this.FindDistanceFromPoints({x:midX, y:midY}, {x:points[point].x, y:points[point].y});
            this.pointsAng.push(ang.angle);
            this.pointsDist.push(dist);
        }
        for(var point =0; point < points.length; point++)
        {
            if ((moX >= points[point].x-15 && moX <= points[point].x+15) && (moY >= points[point].y-15 && moY <= points[point].y+15)) {
                _this.shapeTouchPoint = point;
                this.DownX = moX;
                this.DownY = moY;
            }
        }
    },
    onTouchMove:function(event, canvas){
        if (!event)event=window.event;
        var moX, moY;
        moX = _this.touchCoordsFromEvent(event, false)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, false)[0].y - _this.canvas.offset().top;
        
        if (_this.shapeTouchPoint!=-1) {
            var points = [];
            for (var shape=0; shape<_this.shapes.length;shape++) {
                if (_this.shapes[shape].name == _this.selectedShape) {
                    points = _this.shapes[shape].points;
                    break;
                }
            }
            
            if (_this.selectedShape == "trapezoid") {
                if (_this.shapeTouchPoint==0) {
                    var ang = this.FindAngleFromPoints(points[0], points[1]);
                    ang.angle += 180;
                    var dist = this.FindDistanceFromPoints(points[2], points[3]);
                        points[0].x = moX;
                        points[0].y = moY;
                        points[3].x = points[2].x + dist * Math.cos(ang.angle * (Math.PI / 180));
                        points[3].y = points[2].y + dist * Math.sin(ang.angle * (Math.PI / 180));
                    
                }
                else if (_this.shapeTouchPoint==1) {
                    var ang = this.FindAngleFromPoints(points[1], points[0]);
                    ang.angle += 180;
                    var dist = this.FindDistanceFromPoints(points[3], points[2]);
                    points[1].x = moX;
                    points[1].y = moY;
                    points[2].x = points[3].x + dist * Math.cos(ang.angle * (Math.PI / 180));
                    points[2].y = points[3].y + dist * Math.sin(ang.angle * (Math.PI / 180));
                    
                }
                else if (_this.shapeTouchPoint==2) {
                    var ang = this.FindAngleFromPoints(points[2], points[3]);
                    ang.angle += 180;
                    var dist = this.FindDistanceFromPoints(points[1], points[0]);
                    points[2].x = moX;
                    points[2].y = moY;
                    points[1].x = points[0].x + dist * Math.cos(ang.angle * (Math.PI / 180));
                    points[1].y = points[0].y + dist * Math.sin(ang.angle * (Math.PI / 180));
                    
                }
                else if (_this.shapeTouchPoint==3) {
                    var ang = this.FindAngleFromPoints(points[3], points[2]);
                    ang.angle += 180;
                    var dist = this.FindDistanceFromPoints(points[1], points[0]);
                    points[3].x = moX;
                    points[3].y = moY;
                    points[0].x = points[1].x + dist * Math.cos(ang.angle * (Math.PI / 180));
                    points[0].y = points[1].y + dist * Math.sin(ang.angle * (Math.PI / 180));
                    
                }
                _this.clearAll();
                _this.drawShape();
                _this.showAngle();
                resetEnable();
                this.DownX = moX;
                this.DownY = moY;
            }
            
        }
    },
    onTouchEnd:function(event){
        if (!event)event=window.event;
        console.log(event);
        var moX, moY;
        moX = _this.touchCoordsFromEvent(event, false)[0].x - _this.canvas.offset().left;
        moY = _this.touchCoordsFromEvent(event, false)[0].y - _this.canvas.offset().top;
        
        _this.shapeTouchPoint = -1;
    },
    executeFunction:function(funcObj){
        funcObj.executefunc();
    },
    touchCoordsFromEvent:function(event, isDown){
        var touches = [];
        if (isDown) {
            if (this.isTouchDevice)
                for (var tInd = 0; tInd < event.touches.length; tInd++)
                    touches[tInd] = { x: event.touches[tInd].pageX, y: event.touches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        else{
            if (this.isTouchDevice)
                for (var tInd = 0; tInd < event.changedTouches.length; tInd++)
                    touches[tInd] = { x: event.changedTouches[tInd].pageX, y: event.changedTouches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        return touches;
    },
    initializeDashedLine:function(ctx){
        
        if (!ctx.dashLine) {
            
         ctx.dashedLine = function(x1,y1,x2,y2) {
          var dashStyle = ctx.dashStyle,
           dashCount = dashStyle.length,
           sign = x2>=x1 ? 1 : -1,
              dx = x2-x1,
              dy = y2-y1,
              m = dy/dx,
              xsteps = dashStyle.map(function(len){return sign*Math.sqrt((len*len)/(1 + (m*m)));}),
              dRem =  Math.sqrt( dx*dx + dy*dy ),
              dIndex=0,
              draw=true;
          ctx.moveTo(x1,y1) ;
          while (dRem>=0.1){
                var dLen = dashStyle[dIndex],
                    xStep = xsteps[dIndex];
                if (dLen > dRem) {
                 xStep =  Math.sqrt(dRem*dRem/(1+m*m));
                }
                x1 += xStep ;
                y1 += m*xStep;
                ctx[draw ? 'lineTo' : 'moveTo'](x1,y1);
                dRem -= dLen;
                draw = !draw;
                dIndex = (dIndex+1) % dashCount ;
          }
         };
        }
       
    },
    FindDistanceFromPoints:function(p1, p2){
        return Math.sqrt(Math.pow(p2.y - p1.y, 2)+ Math.pow(p2.x - p1.x, 2));
    },
    FindAngleFromPoints:function (p1, p2){
        var angleDeg = Math.atan2(p2.y - p1.y, p2.x - p1.x) * 180 / Math.PI;
        var angleRadians = (angleDeg) * (Math.PI / 180)
        return {angle:angleDeg, radian:angleRadians};
    },
    FindAngleFrom3Points:function(A, B, C){
        /*
        * Calculates the angle ABC (in degrees) 
        *
        * A first point
        * C second point
        * B center point
        */
        var AB = Math.sqrt(Math.pow(B.x-A.x,2)+ Math.pow(B.y-A.y,2));    
        var BC = Math.sqrt(Math.pow(B.x-C.x,2)+ Math.pow(B.y-C.y,2)); 
        var AC = Math.sqrt(Math.pow(C.x-A.x,2)+ Math.pow(C.y-A.y,2));
        return Math.acos((BC*BC+AB*AB-AC*AC)/(2*BC*AB)) * 57.2957795;  
    },
    PointInShape: function(nvert, vertx, verty, moX, moY ) {
        //nvert - Number of vertices in the polygon. Whether to repeat the first vertex at the end is discussed below.
        //vertx, verty - Arrays containing the x- and y-coordinates of the polygon's vertices.
        //moX, moY - X- and y-coordinate of the mouse.
        var i, j, c = false;
        for( i = 0, j = nvert-1; i < nvert; j = i++ ) {
            if( ( ( verty[i] > moY ) != ( verty[j] > moY ) ) &&
                ( moX < ( vertx[j] - vertx[i] ) * ( moY - verty[i] ) / ( verty[j] - verty[i] ) + vertx[i] ) ) {
                    c = !c;
            }
        }
        return c;
    },
    FindIntersectionPointFrom4Points:function(point1, point2, point3, point4){
        var x1 = point1.x;
        var y1 = point1.y;
        
        var x2 = point2.x;
        var y2 = point2.y;
        
        var x3 = point3.x;
        var y3 = point3.y;
        
        var x4 = point4.x;
        var y4 = point4.y;
        
        var px = ((x1*y2 - y1*x2)*(x3-x4) - (x1-x2)*(x3*y4 - y3*x4)) / ((x1-x2)*(y3-y4) - (y1-y2)*(x3-x4));
        var py = ((x1*y2 - y1*x2)*(y3-y4) - (y1-y2)*(x3*y4 - y3*x4)) / ((x1-x2)*(y3-y4) - (y1-y2)*(x3-x4));
        
        return {x:px, y:py};
    },
};
