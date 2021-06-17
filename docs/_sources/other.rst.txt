その他の機能
==================================

---------------------------------
テーブル
---------------------------------

=====  =====  =======
A      B      A and B
=====  =====  =======
False  False  False
True   False  False
False  True   False
True   True   True
=====  =====  =======

---------------------------------
脚注
---------------------------------

これは 脚注 [#]_ です。

.. [#] 本文の下の方につける注記

---------------------------------
数式
---------------------------------

数式を有効にするには、``conf.py`` に ``sphinx.ext.mathjax`` という拡張モジュールを追加する必要があります。以下のような感じです。モジュールを有効にすれば、LaTeX で数式を記述することができます。

.. code-block:: python
    :caption: conf.py

    # Add any Sphinx extension module names here, as strings. They can be
    # extensions coming with Sphinx (named 'sphinx.ext.*') or your custom
    # ones.
    extensions = ['sphinx.ext.mathjax'
    ]
    
`N` 個の整数 :math:`d[0], d[1], \dots, d[N-1]` が与えられます。これは数式のサンプルです。

