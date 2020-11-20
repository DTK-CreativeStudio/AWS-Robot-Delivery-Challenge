# AWS Robot Delivery Challenge  
## What are These Stuffs?  

これは[AWS Summit Online](https://aws.amazon.com/jp/summits/2020/)で開催された[AWS Robot Delivery Challenge](https://aws.amazon.com/jp/robot-delivery-challenge/)で使用したファイル一式です。  

## Time Schedule  

- 5月
  - 予選 ***7位**/118チーム*  
  5月の予選ではAWS RoboMaker を使用し、シミュレーション上である地点AからBまでのロボットのタイムを競いました。ここでは上位10％にあたる上位12チームが本線進出のために選考されました。   
[予選結果](https://aws.amazon.com/jp/robot-delivery-challenge/ranking/)  
- 9月
  - 本線 *1位/12チーム*
  - 決勝戦 ***2位**/5チーム*   
  本戦では実機を使用し、オンラインの中継を通じ、ロボットへのデプロイ、及び操作を行いました。  
[決勝戦結果](https://aws.amazon.com/jp/robot-delivery-challenge/final-ranking/)  
### YouTube  
[決勝戦、表彰式](https://www.youtube.com/watch?v=Tvhe4P3MiTU)  

[![](http://img.youtube.com/vi/Tvhe4P3MiTU/0.jpg)](http://www.youtube.com/watch?v=Tvhe4P3MiTU "")  

[本線(第2グループ)](https://youtu.be/wjuNznYEFNg?t=219)  

[![](http://img.youtube.com/vi/wjuNznYEFNg/0.jpg)](http://www.youtube.com/watch?v=wjuNznYEFNg "")  

## Component  
上のディレクトリの中身だけではシミュレーションは動きません。  
- AWS Student アカウント  
- AWS RoboMaker  
- Sample アプリケーションのclone  
等が必要となります。  

それらの導入の方法などは以下の資料をご覧ください。  
[ AWS Robot Delivery Workshop
ロボコンのサンプルアプリケーションで自律走行を試す](https://pages.awscloud.com/rs/112-TZM-766/images/RDC-01_AWS_Summit_Online_2020.pdf)  
[aws-robomaker-sample-application-delivery-challenge](https://github.com/aws-samples/aws-robomaker-sample-application-delivery-challenge)  
[AWS Educate 登録ページ](https://www.awseducate.com/registration#APP_TYPE)  

上記のセットアップが終わると、以下のような画面が出てくると思います。  

<div align="center">
<img width="458" height="250" src="https://github.com/DTK-CreativeStudio/AWS-Robot-Delivery-Challenge/blob/master/Images/image1.png" alt="image" title="image">  <br>
</div>  

私の操作したファイルは`/delivery-challenge-sample/robot_ws/src/turtlebot3/turtlebot3_navigation/param`配下の  
- `costmap_common_params_burger.yaml`  
- `dwa_local_planner_params_burger.yaml`  
- `local_costmap_params.yaml`  

であり上の`Sophisticated`ディレクトリに入っています。  
また他にも以下のファイルが入っており、それらの用途は以下のようになっています。  
- `cordinates.txt`  
マップを回るときに停止する必要のある、黄色の線の座標。私のメモ。  
- `map.pgm`  
  本線、及び決勝戦で使用したロボットに読み込ませるマップです。上記の解説資料ではマッピング(マップの形成)しか触れられていませんが、マップを形成したのちにマップを写真加工アプリを使用して、加工することができます。(私は[GIMP](https://forest.watch.impress.co.jp/library/software/gimp/)を使用しました。)

なお、私の当時の環境は`ROS1 Melodic`でした。

## How To Work?
このロボットはROSというオペレーティング システム上で動いており、iPhoneやiPadに搭載されている**LiDARセンサ**等により、位置情報を把握します。  
このアプリケーションでは、LiDARセンサから取得するデータの範囲の調整、及びナビゲーションの設定等でいろいろ変わってきます。   

パラメータの調整次第では滑らかに壁にぶつかる事なく動いてくれますが、このパラメータ調整が難しく、大会中現在ではインターネット上の文献も少なく、コツコツとパラメータ作成及び、マップ加工を行い、滑らかに動くようにしました。  
私が調整したパラメータと、その内容は以下のようになっています。  

1. `costmap_common_params_burger.yaml`  
このファイルではロボットの道の通りやすさを定義します。  
```yaml
inflation_radius: 0.12 #1.0
cost_scaling_factor: 15 #3.0
```  
これらの数値は、そのロボットが自分を中心にどれくらいの範囲のものを検知し、障害物と見なすのかを表しています。これらの数値をデフォルトより低く設定しているのは、狭い道でも通り抜けれるようにするためです。これらの数値が大きいと確実に狭い道は通り抜けることができません。  

2. `dwa_local_planner_params_burger.yaml`  
このファイルではロボットのスピードと、ナビゲーションの各パラメータの重みを定義します。  
```yaml
  max_trans_vel:  0.22
  min_trans_vel:  0.215 #0.11

  max_rot_vel: 2.84 #2.75
  min_rot_vel: 2.74 #1.37

  acc_lim_x: 3.0 #2.5
  acc_lim_y: 0.0
  acc_lim_theta: 3.5 #3.2 
```
これらはそのロボットの直進及び、カーブにおける走行速度を定義しています。  
max値は本戦で使用した*turtlebot 3*の最高速度を設定しています。

```yaml
  path_distance_bias: 30.0 #32.0
  goal_distance_bias: 20.0 #20.0
  occdist_scale: 0.02
  forward_point_distance: 0.325
  stop_time_buffer: 0.15 #0.2
```
これらはナビゲーションを決定するためのパラメータを定義しています。  
`path_distance_bias`の数値をデフォルトから下げたのは、ナビゲーションの経路に忠実に従って、動くと障害物に接触する可能性があるためです。そのため、そのパラメータの数値を下げることで、ナビゲーション経路から道が少しそれても許容できるようにし、障害物に接触する確率を減らしました。
ただし、下げすぎると、ナビゲーション経路から外れすぎて、的外れなナビゲーションの再設定、走行速度の低下を招く可能性があります。  

3. `local_costmap_params.yaml`  
ここでは、LiDARセンサによる情報の取得方法について定義しています。(たぶん)  
```yaml
resolution: 0.02 #0.05
```
デフォルトより数値を下げることで、デフォルトより解像度が上がっています。(たぶん)

4. mapping方式の変更  
`/delivery-challenge-sample/robot_ws/src/delivery_robot_sample/launch/slam.launch`にLiDARで周りの環境をスキャンする際のマッピング方式が設定できるところがあります。`gmapping`の他に、`cartographer`, `hector`, `karto`, `frontier_exploration`があります。詳しくは[[ROS 1] SLAM](https://emanual.robotis.com/docs/en/platform/turtlebot3/slam/#slam)をご覧ください。  

5. mapの加工  
ロボットはmapを使用して、地形や障害物(建物)を認識し、ナビゲーションを構築します。mapの*黒い*ところは壁(障害物)、*グレー*は何もない、*白*は道として認識されます。  
グレーは「何もない」扱いのため、ロボットは可能であれば避けますが、無理に避けることはありません。よくあたる壁や、狭い道などのところにグレーを使用すると、うまい具合にそこを避けたり、スルスルっと道を通ってくれたりします。
詳しくは[サンプルアプリケーション各機能の内部仕様の概要](https://github.com/aws-samples/aws-robomaker-sample-application-delivery-challenge/blob/master/docs/Detail.md)をご覧ください。  
またマップのダウンロード及び、ダウンロードしたマップの加工後のデータのアップロードは以下のようにして行うことができます。

AWSマネジメントコンソールにてS3を開きます。  
<div align="center">
<img width=width="458" height="250" src="https://github.com/DTK-CreativeStudio/AWS-Robot-Delivery-Challenge/blob/master/Images/image2.png" alt="image" title="image">  <br>
</div>  

`cf~`ではなく`deliverychallenge~`の直近のディレクトリを選択してください。(一番下が最新版です)    
<div align="center">
<img width="458" height="250" src="https://github.com/DTK-CreativeStudio/AWS-Robot-Delivery-Challenge/blob/master/Images/image3.png" alt="image" title="image">  <br>
</div>  

`map_store`を開いてください。  
<div align="center">
<img width="458" height="250" src="https://github.com/DTK-CreativeStudio/AWS-Robot-Delivery-Challenge/blob/master/Images/image4.png" alt="image" title="image">  <br>
</div>  

この`map.pgm`がロボットが作ったマップファイルなので、それを適宜加工します。  
<div align="center">
<img width="458" height="250" src="https://github.com/DTK-CreativeStudio/AWS-Robot-Delivery-Challenge/blob/master/Images/image5.png" alt="image" title="image">  <br>
</div>  

以上のようなパラメータ変更や、マップの加工をするとうまくいけば、ロボットが壁にぶつかったり、狭い道を通れなかったりすることはなくなるので、是非やってみてください。
ただし、以下のような注意事項があります。  
- マップを加工しすぎると、LiDARから把握したデータとマップから計算したロボットの位置と実際のロボットの位置に差異が生じるため、誤作動を起こす。  
- 特にマップを何周も回ると、位置情報に狂いが生じ、マップ上でのナビゲーションの位置と実際の位置が合わなくなり、壁にぶつかったりします。  
- シミュレーションと実物で同じアプケーションで動かしても異なる動きをすることがあります。それらは大抵タイヤのスリップやカーブの遠心力による転倒による誤差などです。  


## 参考文献  
[ROS navigation memo](https://qiita.com/idev_jp/items/579933851d3cf1dfc404)  

[ROS Navigation Tuning Guide](http://kaiyuzheng.me/documents/navguide.pdf)  

http://wiki.ros.org/dwa_local_planner#DWAPlannerROS  

http://wiki.ros.org/dwa_local_planner#Oscillation_Prevention_Parameters  

https://sy-base.com/myrobotics/ros/ros-move_base/  

http://wiki.ros.org/base_local_planner  


編集 by [Yusuke](https://github.com/yusuke-1105)
