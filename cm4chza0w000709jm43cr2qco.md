---
title: "【クソアプリ】瞬き耐久チャレンジを作った"
datePublished: Fri Dec 06 2024 08:42:12 GMT+0000 (Coordinated Universal Time)
cuid: cm4chza0w000709jm43cr2qco
slug: 44cq44kv44k944ki44ox44oq44cr556s44gn6icq5lmf44ob44oj44os44oz44k444ks5l2c44gj44gf
canonical: https://qiita.com/nitaking/items/10e50d56e4733acb577b
tags: tensorflow, reactjs, nextjs, mediapipe, 44kv44k944ki44ox44oq

---

本記事は[クソアプリアドベントカレンダー2024](https://qiita.com/advent-calendar/2024/kuso-app)の記事です！

めでたい10周年ということで参加してみましょう。🎉

ということで、

## 瞬き耐久アプリつくった

瞬き、していますか・・・？ あまりにも無意識にしているので、いざ言われてみるとどうでしょう。 まぶたの重み、目の乾燥、感じませんか・・・？

意識しているかどうかで視界は変わるのです。

そう、瞬きを止めてみましょう！

## できたもの

(急な顔面失礼します)

![CleanShot 2024-12-06 at 16 27 29](https://github.com/user-attachments/assets/7cb6db84-3ea6-4653-b435-256f842e72b7 align="left")

[https://github.com/nitaking/blinking-endurance/b](https://github.com/nitaking/blinking-endurance/b)

## ポイント

* 瞬きの判定が正確ではない
    

構成の解説記事を書こうと思っていたが、生成AIベースで中身を理解することなく実装されてしまったため、生成物から紐解いていく。

## 流れ

1. [v0.dev](http://v0.dev)にて叩きを作る
    
    * `tailwind`ベースでのザックリレイアウトが完成
        
    * `Webcam`の実装方法をゲット
        
    * `tensorflow`を用いた目と瞬き判定方法をゲット
        
2. 瞬きの判定がされないなどの動作を修正(with GPT4o/GPT4o1-mini)
    

## 構成要素

* webcam: `react-webcam`
    
    * ユーザーの顔と瞬きを検出
        
* 機械学習: TensorFlow.js, MediaPipe FaceMesh
    
    * TensorFlow.js
        
        * ブラウザ上での機械学習モデル実行のライブラリ
            
        * 顔認識と瞬き検出
            
    * MediaPipe FaceMesh[^1](https://qiita.com/nemutas/items/6321aeca27492baeeb92)
        
        * 顔のキーポイントを検出する、高度な顔ランドマーク検出モデル
            
        * 目のキーポイントを使用して、瞬きを検出
            
        * Eye Aspect Ratio: 瞬き判定のために目の開閉状態を計算
            
* 瞬き判定
    
    * 目の開閉状態の評価指標。一定の閾値を設定することで判定。
        

## 瞬き判定: Eye Aspect Ratio

目の複数のキーポイント間の距離から計算

```ts
const calculateEAR = (eyePoints) => {
  // EAR = (|p2 - p6| + |p3 - p5|) / (2 * |p1 - p4|)
  const A = distance(eyePoints[1], eyePoints[5]);
  const B = distance(eyePoints[2], eyePoints[4]);
  const C = distance(eyePoints[0], eyePoints[3]);
  return (A + B) / (2.0 * C);
};
```

* distance関数: 2点間のユークリッド距離を計算。
    
* しきい値 (blinkThreshold): EARがこの値以下になると瞬きと判定（通常0.2～0.25）。
    

```ts
const distance = (p1: faceLandmarksDetection.Keypoint, p2: faceLandmarksDetection.Keypoint) => {
    return Math.sqrt(Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2));
  };
```

連続フレーム数 (requiredBlinkFrames): 瞬きを安定して検出するために、連続してEARがしきい値以下であるフレーム数を必要とする。

👉️ こちらは生成AIは瞬き検出の精度向上の為に2フレームを何度も求めてきたが、そうなると20ms目を閉じたときだけしか判定されなかったので、1フレーム(10ms)で仮設定。

## アニメーションフレーム管理

こちらはブラウザの再描画に合わせて瞬きの継続的検出を用いているとのことだが、このやり方でなくてもよいとは思っています。

> * **ループの作成**: detectBlink関数内でrequestAnimationFrameを再帰的に呼び出すことで、リアルタイムに瞬き検出を行います。
>     
> * **キャンセル**: コンポーネントのアンマウント時やチャレンジの終了時にcancelAnimationFrameを呼び出し、ループを停止します。
>     

```javascript
useEffect(() => {
  if (model) {
    requestRef.current = requestAnimationFrame(detectBlink);
  }

  return () => {
    if (requestRef.current) {
      cancelAnimationFrame(requestRef.current);
    }
  };
}, [model]);
```

実装構成ファイルは[こちら](https://github.com/nitaking/blinking-endurance/blob/main/app/page.tsx)。

[https://github.com/nitaking/blinking-endurance/blob/main/app/page.tsx](https://github.com/nitaking/blinking-endurance/blob/main/app/page.tsx)

## さいごに

所要時間は約2時間で、レンダリングエラーを修正することに時間を当て込まれる軍配となりました。

Webcamを使った実装も、TensorFlowを使った実装も経験がなかったため、学びが深いです。 意外と簡単に実装ができると思いつつ、生成AIがなかったら瞬き判定で時間が溶けそうだなという所感でした。

ということで、クソアプリを作るつもりがなんだかそこまでクソでもなく、学ばせて頂いた件でした〜！