<script>
(function() {
  // クッキーから同意状態を確認する関数
  function getCookieConsent() {
    const consent = document.cookie.split('; ').find(row => row.startsWith('cookie_consent='));
    return consent ? consent.split('=')[1] : null;
  }

  // ユーザーの年齢・性別・地域・興味関心を取得
  let user_age = localStorage.getItem('user_age') || "unknown";
  let user_gender = localStorage.getItem('user_gender') || "unknown";
  let user_region = localStorage.getItem('user_region') || "unknown";
  let user_interest = localStorage.getItem('user_interest') || "unknown";

  // セッションIDを一意に生成
  function generateSessionId() {
    return 'sess_' + Math.random().toString(36).substr(2, 9);
  }

  let session_id = localStorage.getItem('tracking_session_id');
  if (!session_id) {
    session_id = generateSessionId();
    localStorage.setItem('tracking_session_id', session_id);
  }

  // クッキー同意が得られている場合のみ地域と興味関心を追加
  const consent = getCookieConsent();
  let additionalData = {};

  // クッキー同意がある場合のみ年齢・性別・地域・興味関心を追加
  if (consent === 'true') {
    additionalData = {
      age: user_age,
      gender: user_gender,
      region: user_region,
      interest: user_interest
    };
  } else {
    // 同意がない場合、情報には「クッキー同意なし」として送信
    additionalData = {
      age: "クッキー同意なし",
      gender: "クッキー同意なし",
      region: "クッキー同意なし",
      interest: "クッキー同意なし"
    };
  }

  // トラッキングデータを送信
  function sendTrackingEvent(eventType, additionalData = {}) {
    fetch("https://app.we-eva.com/version-test/api/1.1/wf/event-tracking", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        tracking_id: "SQNQQWSZCI",  // 固定のトラッキングID
        session_id: session_id,
        event_type: eventType,
        url: window.location.href,
        timestamp: Date.now(),
        user_agent: navigator.userAgent,
        screen_width: window.screen.width,
        screen_height: window.screen.height,
        additional_data: JSON.stringify(additionalData)
      })
    })
    .then(response => response.json())
    .then(data => console.log("Tracking Data:", data))
    .catch(error => console.error("Tracking Error:", error));
  }

  // ページビューイベントを送信
  sendTrackingEvent("page_view", additionalData);

  // ページの滞在時間を計測
  let startTime = Date.now();
  let totalDuration = 0;

  // ページ遷移イベント（同一サイト内でページが変更される場合）を検知
  function trackPageTransition() {
    let duration = Date.now() - startTime + totalDuration;
    sendTrackingEvent("page_transition", { duration: duration });

    // 次のページが表示される前に記録をリセット
    totalDuration = duration;
    startTime = Date.now(); // 新しい開始時間をセット
  }

  // ページが非アクティブになった場合の処理
  document.addEventListener("visibilitychange", function() {
    if (document.hidden) {
      trackPageExit();  // ページが非アクティブになったとき
    }
  });

  // タブを閉じる or ページを移動する直前に滞在時間を送信
  window.addEventListener("beforeunload", trackPageExit);

  // サイトから離れる（ページリロードや他のサイトに移動する）際の処理
  function trackPageExit() {
    let duration = Date.now() - startTime + totalDuration;
    sendTrackingEvent("page_exit", { duration: duration });

    // ページの滞在時間をリセット
    totalDuration = duration;
    startTime = Date.now();
  }

  // 同じサイト内のページ遷移を検知（ページリロードを避ける）
  window.addEventListener("popstate", function() {
    if (window.location.href !== location.href) {
      trackPageTransition();  // ページ遷移時に送信
    }
  });

  // hashchange を使って遷移を検知する場合
  window.addEventListener("hashchange", function() {
    trackPageTransition();  // ページ遷移時に送信
  });

})();
</script>
