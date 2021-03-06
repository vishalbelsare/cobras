Package
-------

Examples can also be found in the examples subdirectory.

Running COBRAS_kShape:

    .. code-block:: python

        import os

        import numpy as np
        from sklearn import metrics

        from cobras_ts.cobras_kshape import COBRAS_kShape
        from cobras_ts.labelquerier import LabelQuerier

        ucr_path = '/home/toon/Downloads/UCR_TS_Archive_2015'
        dataset = 'ECG200'
        budget = 100

        data = np.loadtxt(os.path.join(ucr_path,dataset,dataset + '_TEST'), delimiter=',')
        series = data[:,1:]
        labels = data[:,0]

        clusterer = COBRAS_kShape(series, LabelQuerier(labels), budget)
        clusterings, runtimes, ml, cl = clusterer.cluster()

        print(clusterings)
        print("done")
        print(metrics.adjusted_rand_score(clusterings[-1],labels))


Running COBRAS_DTW:

This uses the dtaidistance package to compute the DTW distance matrix.
Note that constructing this matrix is typically the most time consuming step, and significant speedups can be achieved
by using the C implementation in the dtaidistance package.

    .. code-block:: python

        import os

        import numpy as np
        from dtaidistance import dtw
        from sklearn import metrics

        from cobras_ts.cobras_dtw import COBRAS_DTW
        from cobras_ts.labelquerier import LabelQuerier

        ucr_path = '/home/toon/Downloads/UCR_TS_Archive_2015'
        dataset = 'ECG200'
        budget = 100
        alpha = 0.5
        window = 10

        data = np.loadtxt(os.path.join(ucr_path,dataset,dataset + '_TEST'), delimiter=',')
        series = data[:,1:]
        labels = data[:,0]


        dists = dtw.distance_matrix(series, window=int(0.01 * window * series.shape[1]))
        dists[dists == np.inf] = 0
        dists = dists + dists.T - np.diag(np.diag(dists))
        affinities = np.exp(-dists * alpha)

        clusterer = COBRAS_DTW(affinities, LabelQuerier(labels), budget)
        clusterings, runtimes, ml, cl = clusterer.cluster()
