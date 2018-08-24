# Custom

$$
a = b
$$

| dfgdfg | ertert |
| :--- | :--- |
| ertert | dghdfghdgfh |

```csharp
using System;
using System.Collections;
using UnityEngine.SceneManagement;

namespace Ru.Funreality.Utility.DynamicPreloaderPlugin
{
    using UnityEngine;

    public class DynamicPreloader : MonoBehaviour
    {
        [SerializeField] private int _targetScene = 1;

        [SerializeField] private SceneActivationType _sceneActivation;

        public event Action LoadingSceneStart = delegate { };

        public event Action LoadingSceneComplete = delegate { };

        public event Action ActivateSceneComplete = delegate { };

        public event Action<float> LoadingScenePercentage = delegate(float percentage) { };

        public SceneActivationType SceneActivation { get { return _sceneActivation; } }

        private AsyncOperation _currentAsync;

        public enum SceneActivationType
        {
            Auto,
            Manual
        }

        public void ManualLoad() { LoadScene(); }

        public void ManualActivate()
        {
            if (_currentAsync != null && _currentAsync.progress >= 0.9f)
            {
                _currentAsync.allowSceneActivation = true;
                _currentAsync = null;
            }
        }

        private void Start()
        {
            DontDestroyOnLoad(gameObject);
            SceneManager.sceneLoaded += OnSceneLoaded;
            if (SceneActivation == SceneActivationType.Auto)
            {
                LoadScene();
            }
        }

        private void OnDestroy() { SceneManager.sceneLoaded -= OnSceneLoaded; }

        private void OnSceneLoaded(Scene scene, LoadSceneMode loadSceneMode)
        {
            if (scene.buildIndex == _targetScene)
            {
                OnActivateSceneComplete();
                Destroy(gameObject);
            }
        }

        private void LoadScene()
        {
            switch (_sceneActivation)
            {
                case SceneActivationType.Auto:
                    StartCoroutine(LoadSceneAutoRoutine());
                    break;
                case SceneActivationType.Manual:
                    StartCoroutine(LoadSceneManualRoutine());
                    break;
            }
        }


        private IEnumerator LoadSceneAutoRoutine()
        {
            OnLoadingSceneStart();
            _currentAsync = SceneManager.LoadSceneAsync(_targetScene, LoadSceneMode.Single);
            _currentAsync.allowSceneActivation = false;
            while (_currentAsync.progress < 0.9f)
            {
                OnLoadingScenePercentage(_currentAsync.progress);
                yield return 0;
            }
            OnLoadingSceneComplete();
            _currentAsync.allowSceneActivation = true;
            _currentAsync = null;
        }

        private IEnumerator LoadSceneManualRoutine()
        {
            OnLoadingSceneStart();
            _currentAsync = SceneManager.LoadSceneAsync(_targetScene, LoadSceneMode.Single);
            _currentAsync.allowSceneActivation = false;
            while (_currentAsync.progress < 0.9f)
            {
                OnLoadingScenePercentage(_currentAsync.progress);
                yield return 0;
            }
            OnLoadingSceneComplete();
        }

        protected virtual void OnLoadingSceneStart() { LoadingSceneStart(); }
        protected virtual void OnLoadingSceneComplete() { LoadingSceneComplete(); }
        protected virtual void OnActivateSceneComplete() { ActivateSceneComplete(); }
        protected virtual void OnLoadingScenePercentage(float percentage) { LoadingScenePercentage(percentage); }
    }
}
```

