
GPU 最適化済み VM サイズは、負荷の高いグラフィックスのレンダリングやビデオ編集に特化した仮想マシン向けです。 1 つまたは複数の GPU で利用できます。 この記事では、このグループ内の各サイズのストレージのスループットとネットワーク帯域幅に加え、vCPU、データ ディスク、NIC の数に関する情報を提供します。 


NC と NV サイズは、GPU 対応インスタンスとも呼ばれます。 これらは、NVIDIA の GPU カードを含む特殊な仮想マシン サイズで、さまざまなシナリオとユース ケース用に最適化されています。 NV サイズは、リモートの視覚化、ストリーミング、ゲーム、エンコーディング、および OpenGL や DirectX などのフレームワークを使用する VDI シナリオ用に最適化されています。 NC サイズは、CUDA や OpenCL ベースのアプリケーションやシミュレーションなどの、コンピューティング集中型およびネットワーク集中型のアプリケーション、アルゴリズム用にさらに最適化されています。 


NV インスタンスは、NVIDIA の Tesla M60 GPU カードおよびデスクトップ アクセラレータ アプリケーションや仮想デスクトップ向けの NVIDIA GRID を備えていて、お客様は、データやシミュレーションを視覚化することができます。 NV インスタンスでは、グラフィックス処理を要するワークフローを視覚化して優れたグラフィックス機能を活用し、さらにエンコードやレンダリングなどの単精度のワークロードを実行することもできます。 Tesla M60 は、1080p H.264 のストリームを最大 36 備えたデュアル GPU 設計の 4096 CUDA コアを提供します。 

NC インスタンスは NVIDIA の Tesla K80 カードを備えています。 エネルギー調査アプリケーション向け CUDAやクラッシュ シミュレーション、レイ トレーシング レンダリング、深層学習などを活用することで、データをさらに高速に処理できるようになりました。 Tesla K80 は、倍精度で最大 2.91 テラフロップ、単精度で最大 8.93 テラフロップのパフォーマンスを実現する、デュアル GPU 設計の 4992 CUDA コアを提供します。

## <a name="nv-instances"></a>NV インスタンス

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 |
| --- | --- | --- | --- | --- | --- |
| Standard_NV6 |6 |56 |380 | 1 | 24 |
| Standard_NV12 |12 |112 |680 | 2 | 48 |
| Standard_NV24 |24 |224 |1440 | 4 | 64 |

1 GPU = M60 カードの 2 分の 1 相当。

## <a name="nc-instances"></a>NC インスタンス

| サイズ | vCPU | メモリ: GiB | 一時ストレージ (SSD) GiB | GPU | 最大データ ディスク数 |
| --- | --- | --- | --- | --- | --- |
| Standard_NC6 |6 |56 | 380 | 1 | 24 |
| Standard_NC12 |12 |112 | 680 | 2 | 48 |
| Standard_NC24 |24 |224 | 1440 | 4 | 64 |
| Standard_NC24r* |24 |224 | 1440 | 4 | 64 |

1 GPU = K80 カードの 2 分の 1 相当。

* RDMA 対応


