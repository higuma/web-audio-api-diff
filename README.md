# Web Audio APIの仕様変遷調査

2015-03-28時点

Web Audio APIはまだ仕様確定しておらず、仕様の変更や追加・廃止が頻繁に行われている。そこでW3C仕様書から(Web) IDLの部分を抽出して比較し調査した。

## 仕様書のバージョン経緯

まずWeb Audio API draft仕様書のバージョンを確認する。仕様書冒頭部の"Previous version"の部分を順に辿ると、20111215が最初の公開バージョンであることが分かる(リンク先がnoneになる)。それ以降を対象とする。

* [W3C Working Draft 15 December 2011 (最初の公開版)](http://www.w3.org/TR/2011/WD-webaudio-20111215/)
* [W3C Working Draft 15 March 2012](http://www.w3.org/TR/2012/WD-webaudio-20120315/)
* [W3C Working Draft 02 August 2012](http://www.w3.org/TR/2012/WD-webaudio-20120802/)
* [W3C Working Draft 13 December 2012](http://www.w3.org/TR/2012/WD-webaudio-20121213/)
* [W3C Working Draft 10 October 2013 (現在の正式版)](http://www.w3.org/TR/2013/WD-webaudio-20131010/)
* [W3C Editor's Draft 27 March 2015 (次期バージョン暫定版)](http://webaudio.github.io/web-audio-api/)

> "Draft 10"などと書いてある番号はそのバージョンでの(最終)改訂番号。全体の通し番号とは無関係で大きな意味はない。

まずこれらの仕様書からIDL(interface definition language)の記述を抽出する。今回は"Web IDL"(20120802以前は単に"IDL")と書かれているコードブロックの部分を手動で抜き出してテキストファイルを作成した。

> もっとたくさんあれば当然HTMLパーサを使うところだが、今回は数が少ないので処理スクリプト(2-3時間もあれば多分作れる)を作成するよりエディタで手動処理した方が早い。実際の作業は1時間も掛からなかった。

次に時間的に隣り合う版のIDLを2つ見比べて差分のリストを作り、そこから要点をポイントにまとめて要約を作成した。先に要約を示し、最後に差分リストを示す。

## Summary

それぞれのバージョンは一つ前とどこが異なるか順番に要約する。最新バージョンからはじめ、順番に過去に辿っていく。

### 20150327 vs 20131010

最新暫定仕様を実装したブラウザは(たぶん)まだない(現在は作業中で発表しているものはないはず)。以下概略のみ示すが、これらはまだ採用可否不明の暫定仕様だという点に留意すること(2015-03-28時点)。

* AudioContext
    * 非同期処理にcallbackだけでなくPromiseを使えるよう全体的に変更
    * suspend/resume/closeの機能を追加
* AudioNode
    * disconnectを特定ノードだけ選択して切断できるよう改良(今は全切断のみ)
* AudioBuffer
    * チャンネル間のコピー機能を追加(copyFromChannel/copyToChannel)
* AudioBufferSourceNode
    * 新パラメータのdetuneを追加
* AudioWorkerNodeを追加
    * この補助用にAudioWorkerGlobalScopeとAudioProcessEventも追加
* ScriptProcessorNodeとAudioProcessingEventがDEPRECATEDになった
    * 理由はAudioWorkerができたため
    * ScriptProcessorはDOMと同じ空間でコード実行する
        * そのためGUI操作中に音が途切れる現象をどうしても回避できない
    * Workerを使えば別空間なので並列実行できる(音は途切れない)
* StereoPannerを追加
    * 今までのPannerはゲーム向きで、通常のオーディオ用には不便だった
    * StereoPannerはステレオ定位を簡単に設定できる

### 20131010 vs 20121213

現在の最新ブラウザの仕様はここに該当する。

* AudioContext
    * activeSourceCount属性(再生中のサンプルバッファ数モニタ)を廃止
    * createAudioBufferメソッドからArrayBufferを引数に取る形式を廃止
    * createMediaStreamDestination()を追加
    * オシレータ波形生成メソッド名がcreateWaveTableからcreatePeriodicWaveに変更
* OfflineAudioContext
    * 処理をコールバックからイベントハンドラに変更(実質的にはほぼ同じ)
* AudioNode
    * (重要) チャンネル数の取得・設定方法を全面変更
    * これに伴いchannelCount/channelCountMode/channelInterpretationを追加
* AudioDestinationNode
    * チャンネル数の取得・設定方法の変更に伴い全面変更
        * maxNumberOfChannelsはmaxChannelCountに名称変更
        * (AudioNodeにchannelCountができたため)numberOfChannelsは廃止
* AudioParam
    * computedValueを廃止
    * minValueとMaxValueも廃止
* AudioBufferSourceNode
    * playbackState属性を廃止
    * startとstopの第一引数を省略可能に変更(デフォルト値を0として実行)
    * onendedイベントハンドラを追加
* AudioProcessingEvent
    * ScriptProcessorのハンドラ引数仕様からnode属性を削除
* Panner
    * panningModelの値から"soundfield"を廃止
        * 推測するとブラウザに実装されず実情に即してなかったため
        * しかしその後現在まで(5.1等の)音場設定用仕様は未定状態のまま
* WaveShaperNode
    * oversample機能(属性)を追加
* OscillatorNode
    * playbackState属性を廃止
    * 波形設定メソッド名をsetWaveTableからsetPeriodicWaveに変更
    * onendedイベントハンドラを追加
* MediaStreamAudioDestinationNodeを追加

### 20121213 vs 20120802

ここはAPIの名称変更が多く、これが理由で現在のブラウザで過去のコードが動かないケースが多い。特にノード生成API名とnoteOn/noteOffからstart/stopへのメソッド名変更に注意。

* AudioContext
    * ノード生成API名を変更(次の2つ)
        * (重要) createJavaScriptNode(...) → createScriptProcessor(...)
        * (重要) createGainNode() → createGain()
* OfflineAudioContextを追加
* AudioParam
    * computedValueを廃止
* AudioBufferSourceNode (重要、ほぼ全面変更)
    * loopStart, loopEndを導入
    * API名を全面変更
        * noteOn, noteGrainOn, noteOffを廃止
        * 代わりにstart, stopを導入
* Panner
    * panningModelとdistanceModelの値の型を整数から文字列に変更
* BiquadFilter
    * typeの型を整数から文字列に変更
    * (重要) API名をnoteOn, noteOffからstart, stopに変更

### 20120802 vs 20120315

ここから先は初期バージョンと見なし概略だけ示す。

> 当時のブラウザ実装は今はもう探さないと実機確認が困難。仕様書比較で判別できた項目のみ示す。

* AudioContext
    * activeSourceCount属性を導入(しかし後に20131010で廃止)
    * ノード生成APIを追加
        * createMediaElementSource
        * createMediaStreamSource
        * createOscillator(+ createWaveTable)
* AudioNode
    * connectの接続先としてAudioParamを設定できるようになった
* AudioDestinationNode
    * maxNumberOfChannels属性を追加(後に20131010で名称変更)
* AudioParam
    * name, unitsを廃止
* AudioBuffer
    * gainを廃止
* AudioBufferSourceNode
    * gainを廃止
    * playbackState属性を追加(しかし後に20131010で廃止)
* AudioListener
    * gainを廃止
* Oscillatorを追加
    * それに伴いWaveTableも追加
* MediaStreamAudioSourceNodeを追加

### 20120315 vs 20111215

* AudioContext
    * createDelayNodeはオプション引数に最大ディレイ時間を渡せるように改良
* ConvolverNode
    * normalize属性を追加
* DynamicCompressorNodeを(事実上)追加(前の版は仕様が未定だった)
* BiquadFilterNode
    * getFrequencyResponseメソッドを追加

## Whole listing

以下は仕様書のIDLを最新から順にひとつ前のバージョンと比較した変更の全リスト(詳細確認用として保存)。次の方針で作成した。

* メソッド名及び属性名の追加、変更、廃止はすべて記録する
* interfaceは名前が変更されていても機能上同じ場合は記録しない
* enumなどは値の変更があった場合のみ記録

### 20150327 vs 20131010

* `interface AudioContext`
    * added `readonly attribute AudioContextState state`
    * added `Promise<void> suspend()`
    * added `Promise<void> resume()`
    * added `Promise<void> close()`
    * added `attribute EventHandler onstatechange`
    * changed `decodeAudioData(...)`
        * before `void decodeAudioData(...)`
            * `ArrayBuffer audioData`
            * `DecodeSuccessCallback successCallback`
            * `optional DecodeErrorCallback errorCallback`
        * after `Promise<AudioBuffer> decodeAudioData(...)`
            * `ArrayBuffer audioData`
            * `optional DecodeSuccessCallback successCallback`
            * `optional DecodeErrorCallback errorCallback`
    * added `AudioWorkerNode createAudioWorker(...)`
        * `DOMString scriptURL`
        * `optional unsigned long numberOfInputChannels = 2`
        * `optional unsigned long numberOfOutputChannels = 2`
    * added `StereoPannerNode createStereoPanner()`
* `interface OfflineAudioContext`
    * changed `startRendering()`
        * before `void startRendering()`
        * after `Promise<AudioBuffer> startRendering()`
    * added `attribute EventHandler oncomplete`
* `interface AudioNode`
    * changed `disconnect(...)`
        * before `void disconnect(optional unsigned long output = 0)`
        * after
            * `void disconnect()`
            * `void disconnect(unsigned long output)`
            * `void disconnect(AudioNode destination)`
            * `void disconnect(...)`
                * `AudioNode destination`
                * `unsigned long output`
                * `optional unsigned long input`
            * `void disconnect(AudioParam destination)`
            * `void disconnect(AudioParam destination, unsigned long output)`
* `interface AudioBuffer`
    * added `void copyFromChannel(...)`
        * `Float32Array destination`
        * `long channelNumber`
        * `optional unsigned long startInChannel = 0`
    * added `void copyToChannel(...)`
        * `Float32Array source`
        * `long channelNumber`
        * `optional unsigned long startInChannel = 0`
* `interface AudioBufferSourceNode`
    * added `readonly attribute AudioParam detune`
* added `interface AudioWorkerNode`
    * `void terminate()`
    * `void postMessage (any message, optional sequence<Transferable> transfer)`
    * `attribute EventHandler onmessage`
    * `AudioParam addParameter(DOMString name, optional float defaultValue)`
    * `void removeParameter(DOMString name)`
* added `interface AudioWorkerGlobalScope`
    * `attribute EventHandler onaudioprocess`
    * `readonly attribute float sampleRate`
* added `interface AudioProcessEvent`
    * `readonly attribute double playbackTime`
    * `readonly attribute Float32Array[] inputBuffers`
    * `readonly attribute Float32Array[] outputBuffers`
    * `readonly attribute object parameters`
* added `interface StereoPannerNode`
    * `readonly  attribute AudioParam pan`
* `interface AnalyserNode`
    * added `void getFloatTimeDomainData(Float32Array array)`

### 20131010 vs 20121213

* `interface AudioContext`
    * before `interface AudioContext`
    * after `interface AudioContext : EventTarget`
    * removed `readonly attribute unsigned long activeSourceCount`
    * removed `AudioBuffer createBuffer(ArrayBuffer buffer, boolean mixToMono)`
    * added `MediaStreamAudioDestinationNode createMediaStreamDestination()`
* `interface OfflineAudioContext`
    * changed `oncomplete`
        * before `attribute OfflineRenderSuccessCallback oncomplete`
        * after `attribute EventHandler oncomplete`
* added `interface OfflineAudioCompletionEvent`
    * `readonly attribute AudioBuffer renderedBuffer`
* `interface AudioNode`
    * before `interface AudioNode`
    * after `interface AudioNode : EventTarget`
    * added `attribute unsigned long channelCount`
    * added `attribute ChannelCountMode channelCountMode`
    * added `attribute ChannelInterpretation channelInterpretation`
* `interface AudioDestinationNode`
    * removed `readonly attribute unsigned long maxNumberOfChannels`
    * removed `attribute unsigned long numberOfChannels`
    * added `readonly attribute unsigned long maxChannelCount`
* `interface AudioParam`
    * removed `readonly attribute float computedValue`
    * removed `readonly attribute float minValue`
    * removed `readonly attribute float maxValue`
* `interface AudioBufferSourceNode`
    * removed `readonly attribute unsigned short playbackState`
    * changed `void start(...)`
        * before
            * `double when`
            * `optional double offset = 0`
            * `optional double duration`
        * after
            * `optional double when = 0`
            * `optional double offset = 0`
            * `optional double duration`
    * changed `void stop(...)`
        * before `void stop(double when)`
        * after `void stop(optional double when = 0)`
    * added `attribute EventHandler onended`
* `interface AudioProcessingEvent`
    * removed `attribute ScriptProcessorNode node`
* `enum PanningModelType`
    * removed `"soundfield"`
* `interface ConvolverNode`
    * before `attribute AudioBuffer buffer`
    * after `attribute AudioBuffer? buffer`
* `interface WaveShaperNode`
    * added `attribute OverSampleType oversample`
* `interface OscillatorNode`
    * removed `readonly attribute unsigned short playbackState`
    * added `attribute EventHandler onended`
    * changed
        * before `void setWaveTable(WaveTable waveTable)`
        * after `void setPeriodicWave(PeriodicWave periodicWave)`
    * added `attribute EventHandler onended`
* `interface WaveTable`
    * before `interface WaveTable`
    * after `interface PeriodicWave`
* added `interface MediaStreamAudioDestinationNode`
    * `readonly attribute MediaStream stream`

### 20121213 vs 20120802

* `interface AudioContext`
    * changed
        * before `JavaScriptAudioNode createJavaScriptNode(...)`
        * after `ScriptProcessorNode createScriptProcessor(...)`
            * (same arguments)
    * changed
        * before `AudioGainNode createGainNode()`
        * after `GainNode createGain()`
* added `interface OfflineAudioContext`
    * `void startRendering()`
    * `attribute OfflineRenderSuccessCallback oncomplete`
* `interface AudioParam`
    * added `readonly attribute float computedValue`
* `interface AudioBufferSourceNode`
    * added `attribute double loopStart`
    * added `attribute double loopEnd`
    * changed
        * before `void noteOn(double when)`
        * before `void noteGrainOn(...)`
            * `double when`
            * `double grainOffset`
            * `double grainDuration`
        * after `void start(...)`
            * `double when`
            * `optional double offset = 0`
            * `optional double duration`
    * changed
        * before `void noteOff(double when)`
        * after `void stop(double when)`
* `interface ScriptProcessorNode`
    * before `interface JavaScriptAudioNode`
    * after `interface ScriptProcessorNode`
* `interface PannerNode`
    * before `AudioPannerNode`
    * after `PannerNode`
    * changed
        * before `attribute unsigned short panningModel`
        * after `attribute PanningModelType panningModel`
    * changed
        * before `attribute unsigned short distanceModel`
        * after `attribute DistanceModelType distanceModel`
    * removed `readonly attribute AudioGain coneGain`
    * removed `readonly attribute AudioGain distanceGain`
* `interface BiquadFilterNode`
    * changed
        * before `attribute unsigned short type`
        * after `attribute BiquadFilterType type`
* `interface OscillatorNode`
    * changed
        * before `attribute unsigned short type`
        * after `attribute OscillatorType type`
    * changed
        * before `void noteOn(double when)`
        * after `void start(double when)`
    * changed
        * before `void noteOff(double when)`
        * after `void stop(double when)`

### 20120802 vs 20120315

* `interface AudioContext`
    * added `readonly attribute unsigned long activeSourceCount`
    * added `MediaElementAudioSourceNode createMediaElementSource(...)`
        * `HTMLMediaElement mediaElement`
    * added `MediaStreamAudioSourceNode createMediaStreamSource(...)`
        * `MediaStream mediaStream`
    * changed
        * before `AudioChannelSplitter createChannelSplitter()`
        * after `AudioChannelSplitter createChannelSplitter(...)`
            * `[Optional] unsigned long numberOfOutputs = 6`
    * changed
        * before `AudioChannelMerger createChannelMerger()`
        * after `AudioChannelMerger createChannelMerger(...)`
            * `[Optional] unsigned long numberOfInputs = 6`
    * added `Oscillator createOscillator()`
    * added `WaveTable createWaveTable(Float32Array real, Float32Array imag)`
* `interface AudioNode`
    * added `void connect(...)` to AudioParam
        * `AudioParam destination`
        * `[Optional] unsigned long output = 0`
* `interface AudioDestinationNode`
    * added `readonly attribute unsigned long maxNumberOfChannels`
* `interface AudioParam`
    * removed `readonly attribute DOMString name`
    * removed `readonly attribute short units`
* `interface AudioBuffer`
    * removed `attribute AudioGain gain`
* `interface AudioBufferSourceNode`
    * removed `readonly attribute AudioGain gain`
    * added `readonly attribute unsigned short playbackState`
* `interface AudioListener`
    * removed `attribute float gain`
* added `interface Oscillator`
    * `attribute unsigned short type`
    * `readonly attribute unsigned short playbackState`
    * `readonly attribute AudioParam frequency`
    * `readonly attribute AudioParam detune`
    * `void noteOn(in double when)`
    * `void noteOff(in double when)`
    * `void setWaveTable(in WaveTable waveTable)`

* added `interface WaveTable`
* added `interface MediaStreamAudioSourceNode`

### 20120315 vs 20111215

* `interface AudioContext`
    * changed
        * before `DelayNode createDelayNode()`
        * after `DelayNode createDelayNode([Optional] double maxDelayTime)`
* `interface ConvolverNode`
    * added `attribute boolean normalize`
* `interface DynamicsCompressorNode`
    * added `readonly attribute AudioParam threshold`
    * added `readonly attribute AudioParam knee`
    * added `readonly attribute AudioParam ratio`
    * added `readonly attribute AudioParam reduction`
    * added `readonly attribute AudioParam attack`
    * added `readonly attribute AudioParam release`
* `interface BiquadFilterNode`
    * added `void getFrequencyResponse(...)`
        * `Float32Array frequencyHz`
        * `Float32Array magResponse`
        * `Float32Array phaseResponse`
