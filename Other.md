# Other

1. Urllib.urlopen .read() looks weird. Too load img use:
   1. `f = urllib.urlopen(url)`
      `io.BytesIO(f.read())`
   2. Use .read( only once)

2. Get random sample from dir:
   1. shuf -n [100] -e [path/for/source]/* | xargs -i mv {} [path/for/target]

3. Get sample with max[smth] from grouped data:
   1. df.groupby(['name'], sort=False)['from_column'].max()

4. Keras ALWAYS want ALL you memory. Even if u set to use only one core or 1 GPU keras takes all memory from all. To avoid user keras >= 2.1 and:

   1. ```
      import tensorflow as tf
      config = tf.ConfigProto()
      config.gpu_options.visible_device_list = '0'
      config.gpu_options.per_process_gpu_memory_fraction = 0.2
      session = tf.Session(config=config)

      from keras.backend.tensorflow_backend import set_session
      set_session(session)
      ```

      ​


