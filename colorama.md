olorama 套件說明文檔（主要擷取自專案描述部分）：
Colorama 專案說明
使 ANSI 轉義字元序列（用於產生彩色的終端機文字和游標定位）能在 MS Windows 下運作。
如果您覺得 Colorama 很有用，請透過 Paypal 贊助作者。非常感謝！
安裝 (Installation)
已在 CPython 2.7、3.7、3.8、3.9 和 3.10 以及 Pypy 2.7 和 3.8 上進行過測試。
除了標準函式庫外，沒有其他相依套件需求。
pip install colorama
# 或者
conda install -c anaconda colorama

描述 (Description)
長久以來，ANSI 轉義字元序列一直被用來在 Unix 和 Mac 上產生彩色的終端機文字和游標定位。Colorama 透過包裝 stdout，剝離它找到的 ANSI 序列（這些序列在輸出中會顯示為亂碼），並將它們轉換為適當的 win32 呼叫以修改終端機狀態，讓這項功能也能在 Windows 上運作。在其他平台上，Colorama 不會執行任何動作。
這樣的結果是提供了一個簡單的跨平台 API，用於從 Python 印出彩色的終端機文字，並帶來了一個令人高興的副作用：現有在 Linux 或 Mac 上使用 ANSI 序列產生彩色輸出的應用程式或函式庫，現在只要呼叫 colorama.just_fix_windows_console()（自 v0.4.6 起）或 colorama.init()（所有版本，但可能有其他副作用 – 見下文），就能直接在 Windows 上運作。
另一種替代方法是在 Windows 機器上安裝 ansi.sys，這能為所有在終端機中執行的應用程式提供相同的行為。Colorama 是為了那些不容易進行此操作的情況而設計的（例如，也許您的應用程式沒有安裝程式）。
原始碼儲存庫中的展示腳本使用 ANSI 序列印出了一些彩色文字。比較它們在 Gnome-terminal 內建的 ANSI 處理下的輸出，與在 Windows 命令提示字元下使用 Colorama 的輸出：
 * 在 Ubuntu 下使用 gnome-terminal 的 ANSI 序列。
  * 在 Windows 上使用 Colorama 的相同 ANSI 序列。
  這些螢幕截圖顯示，在 Windows 上，Colorama 不支援 ANSI 的「暗淡文字 (dim text)」；它看起來和「一般文字」一樣。
  用法 (Usage)
  初始化 (Initialisation)
  如果您使用 Colorama 唯一想要的就是讓 ANSI 轉義序列能在 Windows 上運作，那麼請執行：
  from colorama import just_fix_windows_console
  just_fix_windows_console()

  如果您使用的是較新版本的 Windows 10 或更高版本，且您的 stdout/stderr 指向 Windows 終端機，這將會切換神奇的設定開關以啟用 Windows 內建的 ANSI 支援。如果您使用的是較舊版本的 Windows，且您的 stdout/stderr 指向 Windows 終端機，這會將 sys.stdout 和/或 sys.stderr 包裝在一個神奇的檔案物件中，該物件會攔截 ANSI 轉義序列並發出適當的 Win32 呼叫來模擬它們。
  在所有其他情況下，它什麼也不做。基本上，這個想法是讓 Windows 在處理 ANSI 轉義序列方面表現得像 Unix 一樣。多次呼叫此函式是安全的。在非 Windows 平台上呼叫此函式也是安全的，但它不會執行任何動作。當您的 stdout/stderr 其中之一或兩者被重新導向至檔案時，呼叫此函式也是安全的——它不會對那些串流執行任何動作。
  或者，您可以使用具有更多功能（但也可能有更多潛在陷阱）的舊版介面：
  from colorama import init
  init()

  這與 just_fix_windows_console 的作用相同，除了以下差異：
   * 多次呼叫 init 是不安全的；您最終可能會得到多層包裝和損壞的 ANSI 支援。
    * Colorama 會應用啟發式演算法來猜測 stdout/stderr 是否支援 ANSI，如果它認為不支援，它會將 sys.stdout 和 sys.stderr 包裝在一個神奇的檔案物件中，該物件在印出之前會剝離 ANSI 轉義序列。這發生在所有平台上，如果您希望撰寫的程式碼能無條件發出 ANSI 轉義序列，並讓 Colorama 決定它們是否應該實際輸出，這會很方便。但請注意，Colorama 的啟發式演算法並不是特別聰明。
     * init 也接受明確的關鍵字參數來啟用/停用各種功能 – 見下文。
     如果要在程式結束前停止使用 Colorama，只需呼叫 deinit()。這將會把 stdout 和 stderr 恢復成它們原本的值，從而停用 Colorama。若要再次恢復使用 Colorama，請呼叫 reinit()；它的效能消耗比再次呼叫 init() 更低（但作用相同）。
     大多數使用者應該依賴 colorama >= 0.4.6，並使用 just_fix_windows_console。舊的 init 介面將會無限期地支援以維持向後相容性，但我們不打算修復它的任何問題，同樣也是為了向後相容性。
     彩色輸出 (Colored Output)
     然後可以使用 Colorama 的 ANSI 轉義序列常數簡寫來完成跨平台的彩色文字列印。這些被刻意設計得很基礎，見下文。
     from colorama import Fore, Back, Style
     print(Fore.RED + 'some red text')
     print(Back.GREEN + 'and with a green background')
     print(Style.DIM + 'and in dim text')
     print(Style.RESET_ALL)
     print('back to normal now')

     …或者只是簡單地從您自己的程式碼中手動印出 ANSI 序列：
     print('\033[31m' + 'some red text')
     print('\033[39m') # and reset to default color

     …或者，Colorama 可以與現有的 ANSI 函式庫結合使用，例如歷史悠久的 Termcolor、極好的 Blessings 或令人難以置信的 Rich。如果您希望 Colorama 的 Fore、Back 和 Style 常數能具備更多功能，那麼可以考慮使用上述其中一個功能強大的函式庫來產生顏色等，並僅將 Colorama 用於其主要目的：轉換這些 ANSI 序列讓它們也能在 Windows 上運作：
     同樣地，請不要發送將產生新 ANSI 類型功能加入 Colorama 的 PR。 我們只對將 ANSI 程式碼轉換為 win32 API 呼叫感興趣，而不是像上面那樣用於產生 ANSI 字元的捷徑。
     可用的格式化常數為：
      * Fore: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE, RESET.
       * Back: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE, RESET.
        * Style: DIM, NORMAL, BRIGHT, RESET_ALL
        Style.RESET_ALL 會重置前景色、背景色和亮度。Colorama 會在程式退出時自動執行此重置。
        這些常數有相當好的支援，但並不屬於標準的一部分：
         * Fore: LIGHTBLACK_EX, LIGHTRED_EX... (等等)
          * Back: LIGHTBLACK_EX, LIGHTRED_EX... (等等)
          游標定位 (Cursor Positioning)
          支援重新定位游標的 ANSI 代碼。有關如何產生它們的範例，請參閱 demos/demo06.py。
          Init 關鍵字參數 (Init Keyword Args)
          init() 接受一些 **kwargs 來覆寫預設行為。
           * init(autoreset=False): 如果您發現自己在每次列印結束時都重複發送重置序列來關閉顏色變更，那麼 init(autoreset=True) 將會自動化這個過程。
            * init(strip=None): 傳遞 True 或 False 來覆寫是否應該從輸出中剝離 ANSI 代碼。預設行為是如果在 Windows 上，或者如果輸出被重新導向（非 tty），則進行剝離。
             * init(convert=None): 傳遞 True 或 False 來覆寫是否要將輸出中的 ANSI 代碼轉換為 win32 呼叫。預設行為是如果在 Windows 上且輸出到 tty（終端機），則進行轉換。
              * init(wrap=True): 在 Windows 上，Colorama 透過將 sys.stdout 和 sys.stderr 替換為代理物件來運作，這些物件會覆寫 .write() 方法來執行它們的工作。如果這種包裝給您帶來問題，那麼可以透過傳遞 init(wrap=False) 來停用它。預設行為是如果 autoreset、strip 或 convert 為 True，則進行包裝。當停用包裝時，在非 Windows 平台上的彩色印出將繼續正常運作。若要進行跨平台的彩色輸出，您可以直接使用 Colorama 的 AnsiToWin32 代理。
              已識別的 ANSI 序列 (Recognised ANSI Sequences)
              ANSI 序列通常採用以下形式：
              ESC [ <param> ; [span_65](start_span)<param> ... <command>
              其中 <param> 是一個整數，而 <command> 是一個單一字母。零個或多個參數被傳遞給一個 <command>。如果沒有傳遞參數，它通常等同於傳遞單一的零。
              給 'm' 指令的多個數值參數可以組合成單一序列，例如 ESC [ 36 ; 45 ; [span_68](start_span)1 m 代表洋紅色背景上的明亮青色文字。
              所有其他格式為 ESC [ <param> ; [span_69](start_span)<param> ... <command> 的 ANSI 序列，在 Windows 上都會默默地從輸出中被剝離。任何其他形式的 ANSI 序列，例如單字元代碼或替代的起始字元，都不會被識別或剝離。
              狀態與已知問題 (Status & Known Problems)
              作者個人僅在 Windows XP (CMD, Console2)、Ubuntu (gnome-terminal, xterm) 和 OS X 上測試過。某些有效的 ANSI 序列無法被識別。
              如果您正在修改程式碼，請參閱 README-hacking.md。尤其是，請查看那裡的解釋，說明為什麼我們不希望有能讓 Colorama 產生新型 ANSI 程式碼的 PR。未解決的問題和願望清單可以在專案的 GitHub Issues 中找到。
              授權條款 (License)
              版權所有 Jonathan Hartley & Arnon Yaari, 2013-2020。BSD 3-Clause 授權條款；請參閱 LICENSE 檔案。
              需要我為您進一步解釋這份文件中的任何特定程式碼範例或功能嗎？
              
根據提供的說明文檔，Colorama 的核心功能與特色可以概述如下：
 * 核心功能（Windows 支援）：讓原本用於產生彩色終端機文字與游標定位的 ANSI 轉義字元序列，能夠在 MS Windows 系統上正常運作。
  * 底層運作原理：在 Windows 上，它會包裝標準輸出 (stdout)，攔截並剝離 ANSI 序列（避免顯示為亂碼），然後將其轉換為適當的 win32 API 呼叫來改變終端機狀態。
   * 跨平台相容性：提供了一個簡單的跨平台 API。在 Windows 以外的作業系統（如 Unix 和 Mac）上，Colorama 不會執行任何動作，讓同一套程式碼可以無縫在不同平台上執行。
    * 簡單的初始化：透過呼叫 just_fix_windows_console() 或 init()，就能自動處理環境判定（如啟用 Windows 10 內建支援或使用包裝器模擬）。
     * 第三方函式庫相容：能讓原本設計給 Linux/Mac 的 ANSI 輸出函式庫（如 Termcolor、Blessings 或 Rich）只需搭配 Colorama 就能直接在 Windows 上運作。
      * 內建排版常數：提供基礎且易用的常數（Fore 前景色、Back 背景色、Style 樣式），讓開發者可以直接在 Python 中印出帶有顏色或亮度（如暗淡或明亮）的文字。
       * 游標定位：支援使用 ANSI 代碼在終端機中上下左右移動游標的位置。
       需要我為您列出如何使用它來印出「彩色文字」的具體 Python 程式碼範例嗎？

這裡為您準備幾個 colorama 在實際開發中最常見、最實用的 Python 程式碼應用案例。
在執行這些程式碼之前，請確保您已經安裝了該套件（pip install colorama）。
實用案例一：自動重置顏色的終端機日誌 (Log) 提示
在開發命令列工具時，我們通常會用綠色表示成功，紅色表示錯誤，黃色表示警告。使用 autoreset=True 可以讓您不用每次都手動加上 Style.RESET_ALL，印完一行就會自動變回預設顏色。
from colorama import init, Fore, Style

# 初始化並開啟自動重置功能
init(autoreset=True)

print(Fore.GREEN + "[成功] 資料庫連線已建立。")
print(Fore.YELLOW + "[警告] 找不到設定檔，將使用預設設定。")
print(Fore.RED + Style.BRIGHT + "[錯誤] 檔案寫入失敗！權限不足。")

# 因為有 autoreset=True，這行會自動恢復成一般終端機的預設顏色
print("系統繼續執行其他常規任務...")

實用案例二：製作醒目的命令列標題與選單 (混合樣式)
如果您要寫一個終端機的互動介面，可能會需要一個醒目的標題。您可以同時疊加前景色 (Fore)、背景色 (Back) 和文字亮度 (Style)。
from colorama import init, Fore, Back, Style

init() # 這裡不使用 autoreset，我們手動控制

# 產生白字、藍底、高亮度的標題列
title = Back.BLUE + Fore.WHITE + Style.BRIGHT + " === 系統管理控制台 === " + Style.RESET_ALL
print(title)

print(Fore.CYAN + "1. " + Fore.WHITE + "檢視系統狀態")
print(Fore.CYAN + "2. " + Fore.WHITE + "重新啟動伺服器")
print(Fore.CYAN + "3. " + Fore.RED + "關閉系統" + Style.RESET_ALL)

# 記得在最後重置，以免影響後續使用者的終端機
print(Style.RESET_ALL)

實用案例三：結合 Python 內建的 logging 模組
在大型專案中，我們通常會使用原生的 logging 模組。您可以利用 colorama 來自訂一個 Formatter，讓不同層級的日誌在終端機中呈現不同顏色，方便除錯時一眼看出問題。
import logging
from colorama import init, Fore, Style

init(autoreset=True)

class ColorFormatter(logging.Formatter):
    # 定義不同日誌層級對應的顏色
        COLORS = {
                logging.DEBUG: Fore.BLUE,
                        logging.INFO: Fore.GREEN,
                                logging.WARNING: Fore.YELLOW,
                                        logging.ERROR: Fore.RED,
                                                logging.CRITICAL: Fore.RED + Back.WHITE + Style.BRIGHT,
                                                    }

                                                        def format(self, record):
                                                                log_color = self.COLORS.get(record.levelno, Fore.WHITE)
                                                                        # 設定日誌輸出的格式：時間 - 層級 - 訊息
                                                                                format_str = f"{log_color}%(asctime)s - %(levelname)s - %(message)s"
                                                                                        formatter = logging.Formatter(format_str, datefmt="%H:%M:%S")
                                                                                                return formatter.format(record)

                                                                                                # 設定 Logger
                                                                                                logger = logging.getLogger("MyServer")
                                                                                                logger.setLevel(logging.DEBUG)
                                                                                                ch = logging.StreamHandler()
                                                                                                ch.setFormatter(ColorFormatter())
                                                                                                logger.addHandler(ch)

                                                                                                # 測試輸出
                                                                                                logger.debug("這是一條除錯訊息")
                                                                                                logger.info("伺服器已啟動於 port 8080")
                                                                                                logger.warning("記憶體使用率超過 80%")
                                                                                                logger.error("無法載入使用者資料")
                                                                                                logger.critical("系統崩潰！")

                                                                                                實用案例四：進度或狀態的行內更新 (覆寫同一行文字)
                                                                                                利用 ANSI 的游標控制字元（如 \r 回到行首，配合 \033[K 清除游標到行尾的內容），您可以在終端機的同一行不斷更新進度，而不會瘋狂洗版。colorama 確保這在 Windows 上也能完美運作。
                                                                                                import time
                                                                                                from colorama import init, Fore

                                                                                                # 初始化 colorama
                                                                                                init(autoreset=True)

                                                                                                print("開始下載檔案...")

                                                                                                total_steps = 10
                                                                                                for i in range(total_steps + 1):
                                                                                                    # \r 讓游標回到行首，\033[K 是標準 ANSI 清除行尾的指令
                                                                                                        # 搭配 colorama 可以在 Windows cmd/powershell 正常發揮作用
                                                                                                            progress_msg = f"\r\033[K" + Fore.CYAN + f"進度: [{'#' * i}{'.' * (total_steps - i)}] {i * 10}%"
                                                                                                                
                                                                                                                    # 使用 end="" 避免自動換行，flush=True 強制立即輸出
                                                                                                                        print(progress_msg, end="", flush=True)
                                                                                                                            time.sleep(0.3)

                                                                                                                            print("\n" + Fore.GREEN + "下載完成！")

                                                                                                                            這幾個案例涵蓋了從簡單的 print 到整合標準庫的進階用法。您需要我針對哪一個案例做更深入的解釋，或是想了解如何把它應用到您目前的專案中嗎？
