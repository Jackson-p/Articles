效果无非就是把一个元素比如图片放在一个控制器里，用户可以通过手势任意修改这个组件的位置、大小、角度等。

自己实现的代码如下:(细节实现日后再写)
```js
import React, { Component } from 'react';
import './style.less'


interface Props {

}
interface State {
    x: number;
    y: number;
    s: number;
    r: number;
}

type ContollerStyle = {
    transform: string;
    width: string;
    height: string;
}

export default class ImgController extends Component<Props, State> {
    translate: Array<number> = [0, 0]
    material_middle: Array<number> = [50, 50]
    touch_start_pos: Array<number> = [0, 0]
    scale: number = 1
    rotate: number = 0
    readonly material_size = {
        width: 100,
        height: 100
    }

    state: State = {
        x: 0,
        y: 0,
        s: 1,
        r: 0
    }
    
    handleTouchStart = (event) => {
        const touches = event.touches[0];
        this.touch_start_pos = [touches.pageX, touches.pageY];
    }
    handleTouchMove = (event) => {
        const touches = event.touches[0];
        this.material_middle = [
            this.material_middle[0] + touches.pageX - this.touch_start_pos[0],
            this.material_middle[1] + touches.pageY - this.touch_start_pos[1]
        ];
        this.translate = [
            this.translate[0] + touches.pageX - this.touch_start_pos[0],
            this.translate[1] + touches.pageY - this.touch_start_pos[1]
        ];
        this.touch_start_pos = [touches.pageX, touches.pageY];
        this.setState({
            x: this.translate[0],
            y: this.translate[1]
        });
    }
    handleTouchRotate = (event) => {
        const touches = event.touches[0];
        this.scale = Math.sqrt(Math.pow(touches.pageX - this.material_middle[0], 2) + Math.pow(touches.pageY - this.material_middle[1], 2)) / Math.sqrt(Math.pow(this.material_size.width / 2, 2) + Math.pow(this.material_size.height / 2, 2));
        this.rotate = Math.atan2(touches.pageY - this.material_middle[1], touches.pageX - this.material_middle[0]) * 180 / Math.PI - 45;
        this.setState({
            s: this.scale,
            r: this.rotate
        })
        event.stopPropagation();
    }
    render() {
        let controller_style: ContollerStyle = {
            transform: `translate(${this.state.x}px, ${this.state.y}px) scale(${this.state.s}) rotate(${this.state.r}deg)`,
            width: `${this.material_size.width}px`,
            height: `${this.material_size.height}px`
        }
        return (
            <div className="image-controller" 
                 onTouchStart={this.handleTouchStart}
                 onTouchMove={this.handleTouchMove}
                 style={controller_style}
                >
                <div className="load-img"></div>
                <div className="operator-wrapper">
                    <span className="operator" onTouchMove={this.handleTouchRotate} ></span>
                </div>
            </div>
        )
    }
}
```