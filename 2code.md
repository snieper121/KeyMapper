могу сказать, что в целом код выглядит хорошо структурированным, но есть несколько моментов, которые стоит учесть.

Общий обзор проекта
Ваш проект представляет собой приложение для Android, которое позволяет создавать и настраивать пользовательские маппинги кнопок и жестов для игр. Проект включает:

Сервис доступности (MappingService) для перехватывания ввода
Функционал записи и воспроизведения действий
Настройку кнопок геймпада
Управление профилями для разных игр
Настройку жестов
Пользовательский интерфейс для всех этих функций
Структура проекта
Проект содержит 37 файлов, включая:

Kotlin-файлы с основной логикой
XML-файлы для макетов и конфигурации
Тестовые файлы (RecorderManagerTest.kt, MappingServiceTest.kt)
Положительные стороны
Хорошее разделение обязанностей: Каждый класс выполняет свою конкретную функцию
Наличие тестов: Есть модульные тесты для ключевых компонентов
Корректная структура Android-приложения: Правильно настроены Activity, Service и другие компоненты
Поддержка профилей: Возможность сохранять и загружать различные конфигурации
Расширяемость: Архитектура позволяет легко добавлять новые функции
Моменты, требующие внимания
Отсутствующие компоненты:

ProfileManager указан как отдельный файл, но на самом деле он реализован как объект внутри GameProfile.kt
Потенциальные улучшения:

Можно добавить больше тестов для покрытия всех основных компонентов
Некоторые классы имеют много зависимостей, что может усложнить поддержку кода
Взаимосвязи компонентов
Ваш проект имеет следующие ключевые взаимосвязи:

MainActivity является центральным компонентом, который использует большинство других классов
MappingService взаимодействует с ButtonSettingsDialog, CustomOverlayButton и GameProfile
Компоненты для настройки (EditMappingActivity, GamepadConfigActivity, GestureConfigActivity) используют соответствующие модели данных
Тесты
В проекте есть два тестовых класса:

RecorderManagerTest: 2 тестовых метода, 7 проверок
MappingServiceTest: 1 тестовый метод, 2 проверки
Заключение
Ваш проект GameMapper хорошо структурирован и не содержит явных ошибок в коде. Единственное замечание касается организации ProfileManager, который реализован как объект внутри файла GameProfile.kt, а не как отдельный файл.

Рекомендую:

Рассмотреть возможность вынесения ProfileManager в отдельный файл для лучшей организации кода
Добавить больше тестов для повышения надежности приложения
Продолжить разработку с учетом принципов чистой архитектуры
В целом, проект выглядит работоспособным и готовым к использованию.





В процессе разработки предоставь уже готовый и исправленный код, так же обозначай мы код изменили или добавили новый, также напиши название файлов которые мы не меняли, у них код показывать не нужно, Рассмотреть возможность вынесения ProfileManager в отдельный файл для лучшей организации кода
Добавить больше тестов для повышения надежности приложения
Продолжить разработку с учетом принципов чистой архитектуры

2/2


Я помогу вам улучшить проект, вынести ProfileManager в отдельный файл, добавить тесты и применить принципы чистой архитектуры. Вот план изменений:

1. Вынесение ProfileManager в отдельный файл
Новый файл: ProfileManager.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.util.Log
import android.view.KeyEvent

/**
 * Менеджер профилей для управления игровыми профилями
 * Реализует паттерн Singleton для централизованного управления профилями
 */
object ProfileManager {
    private const val TAG = "ProfileManager"
    private val profiles = mutableMapOf<String, GameProfile>()
    
    /**
     * Инициализация менеджера профилей с профилем по умолчанию
     */
    init {
        // Создаем профиль по умолчани
        val defaultProfile = GameProfile(
            name = "Default Profile"
        ).apply {
            keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
            keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
            keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
            keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
            keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
            keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
            keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
        }
        
        profiles[defaultProfile.id] = defaultProfile
    }
    
    /**
     * Сохраняет профиль в хранилище
     * @param profile Профиль для сохранения
     */
    fun saveProfile(profile: GameProfile) {
        profiles[profile.id] = profile
        PersistenceManager.saveProfiles(profiles.values.toList())
        Log.d(TAG, "Профиль сохранен: ${profile.name}")
    }
    
    /**
     * Получает профиль по идентификатору
     * @param id Идентификатор профиля
     * @return Профиль или null, если не найден
     */
    fun getProfile(id: String): GameProfile? {
        return profiles[id]
    }
    
    /**
     * Получает список всех профилей
     * @return Список всех профилей
     */
    fun getAllProfiles(): List<GameProfile> {
        return profiles.values.toList()
    }
    
    /**
     * Удаляет профиль по идентификатору
     * @param id Идентификатор профиля для удаления
     */
    fun deleteProfile(id: String) {
        val profile = profiles[id]
        profiles.remove(id)
        PersistenceManager.saveProfiles(profiles.values.toList())
        Log.d(TAG, "Профиль удален: ${profile?.name}")
    }
    
    /**
     * Получает профиль для указанного пакета приложения
     * @param packageName Имя пакета приложения
     * @return Профиль для указанного приложения или null
     */
    fun getProfileForPackage(packageName: String): GameProfile? {
        return profiles.values.find { it.packageName == packageName }
    }
    
    /**
     * Загружает профили из хранилища
     */
    fun loadProfiles() {
        val loadedProfiles = PersistenceManager.loadProfiles()
        profiles.clear()
        loadedProfiles.forEach { profiles[it.id] = it }
        
        // Если нет профилей, создаем профиль по умолчанию
        if (profiles.isEmpty()) {
            val defaultProfile = GameProfile(
                name = "Default Profile"
            ).apply {
                keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
                keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
                keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
                keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
                keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
                keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
                keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
            }
            
            profiles[defaultProfile.id] = defaultProfile
            PersistenceManager.saveProfiles(profiles.values.toList())
        }
        
        Log.d(TAG, "Загружено профилей: ${profiles.size}")
    }
    
    /**
     * Создает новый профиль с указанным именем
     * @param name Имя нового профиля
     * @return Созданный профиль
     */
    fun createNewProfile(name: String): GameProfile {
        val profile = GameProfile(name = name)
        saveProfile(profile)
        return profile
    }
    
    /**
     * Дублирует существующий профиль с новым именем
     * @param sourceId Идентификатор исходного профиля
     * @param newName Имя для нового профиля
     * @return Новый профиль или null если исходный профиль не найден
     */
    fun duplicateProfile(sourceId: String, newName: String): GameProfile? {
        val sourceProfile = getProfile(sourceId) ?: return null
        
        val newProfile = GameProfile(
            name = newName,
            packageName = sourceProfile.packageName
        )
        
        // Копируем настройки из исходного профиля
        newProfile.keyMappings.putAll(sourceProfile.keyMappings)
        newProfile.gestureMappings.addAll(sourceProfile.gestureMappings)
        newProfile.buttonSettings.putAll(sourceProfile.buttonSettings)
        
        saveProfile(newProfile)
        return newProfile
    }
}
Измененный файл: GameProfile.kt
kotlin

Copy
package com.example.gamemapper

import android.view.KeyEvent
import java.util.UUID

/**
 * Модель данных для игрового профиля
 */
data class GameProfile(
    val id: String = UUID.randomUUID().toString(),
    var name: String,
    var packageName: String = "",
    val keyMappings: MutableMap<Int, Pair<Float, Float>> = mutableMapOf(),
    val gestureMappings: MutableList<GestureMapping> = mutableListOf(),
    val buttonSettings: MutableMap<Int, ButtonSettings> = mutableMapOf()
)

/**
 * Настройки кнопки для профиля
 */
data class ButtonSettings(
    val shape: Int = CustomOverlayButton.SHAPE_CIRCLE,
    val color: Int = android.graphics.Color.parseColor("#80FFFFFF"),
    val size: Float = 120f,
    val alpha: Int = 128,
    val borderColor: Int = android.graphics.Color.BLACK,
    val borderWidth: Float = 0f
)

/**
 * Модель данных для маппинга жестов
 */
data class GestureMapping(
    val id: String = UUID.randomUUID().toString(),
    val gestureType: GestureType,
    val keyCode: Int,
    val startX: Float,
    val startY: Float,
    val endX: Float? = null,
    val endY: Float? = null,
    val duration: Long = 0,
    val tapCount: Int = 1,
    val radius: Float = 0f
)

/**
 * Типы поддерживаемых жестов
 */
enum class GestureType {
    TAP, LONG_PRESS, SWIPE, MULTI_TAP, CIRCLE
}
2. Добавление новых тестов
Новый файл: ProfileManagerTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.view.KeyEvent
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito.*

class ProfileManagerTest {
    
    private lateinit var context: Context
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        // Мокируем PersistenceManager для тестирования
        val mockPersistenceManager = mock(PersistenceManager::class.java)
        `when`(mockPersistenceManager.loadProfiles()).thenReturn(emptyList())
    }
    
    @Test
    fun testCreateNewProfile() {
        val profileName = "Test Profile"
        val profile = ProfileManager.createNewProfile(profileName)
        
        assertEquals(profileName, profile.name)
        assertTrue(ProfileManager.getAllProfiles().contains(profile))
    }
    
    @Test
    fun testGetProfileForPackage() {
        val packageName = "com.example.testapp"
        val profile = ProfileManager.createNewProfile("Package Test")
        profile.packageName = packageName
        ProfileManager.saveProfile(profile)
        
        val retrievedProfile = ProfileManager.getProfileForPackage(packageName)
        assertNotNull(retrievedProfile)
        assertEquals(profile.id, retrievedProfile?.id)
    }
    
    @Test
    fun testDeleteProfile() {
        val profile = ProfileManager.createNewProfile("Delete Test")
        val profileId = profile.id
        
        assertTrue(ProfileManager.getAllProfiles().any { it.id == profileId })
        
        ProfileManager.deleteProfile(profileId)
        
        assertFalse(ProfileManager.getAllProfiles().any { it.id == profileId })
        assertNull(ProfileManager.getProfile(profileId))
    }
    
    @Test
    fun testDuplicateProfile() {
        val originalName = "Original Profile"
        val duplicateName = "Duplicate Profile"
        
        val originalProfile = ProfileManager.createNewProfile(originalName)
        originalProfile.keyMappings[KeyEvent.KEYCODE_A] = Pair(100f, 200f)
        originalProfile.keyMappings[KeyEvent.KEYCODE_B] = Pair(300f, 400f)
        ProfileManager.saveProfile(originalProfile)
        
        val duplicatedProfile = ProfileManager.duplicateProfile(originalProfile.id, duplicateName)
        
        assertNotNull(duplicatedProfile)
        assertEquals(duplicateName, duplicatedProfile?.name)
        assertEquals(originalProfile.keyMappings.size, duplicatedProfile?.keyMappings?.size)
        assertEquals(
            originalProfile.keyMappings[KeyEvent.KEYCODE_A], 
            duplicatedProfile?.keyMappings?.get(KeyEvent.KEYCODE_A)
        )
    }
}
Новый файл: GestureMappingTest.kt
kotlin

Copy
package com.example.gamemapper

import android.view.KeyEvent
import org.junit.Assert.*
import org.junit.Test

class GestureMappingTest {
    
    @Test
    fun testTapGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.TAP,
            keyCode = KeyEvent.KEYCODE_A,
            startX = 100f,
            startY = 200f
        )
        
        assertEquals(GestureType.TAP, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_A, gesture.keyCode)
        assertEquals(100f, gesture.startX)
        assertEquals(200f, gesture.startY)
        assertNull(gesture.endX)
        assertNull(gesture.endY)
        assertEquals(0L, gesture.duration)
        assertEquals(1, gesture.tapCount)
        assertEquals(0f, gesture.radius)
    }
    
    @Test
    fun testSwipeGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.SWIPE,
            keyCode = KeyEvent.KEYCODE_B,
            startX = 100f,
            startY = 200f,
            endX = 300f,
            endY = 400f,
            duration = 500L
        )
        
        assertEquals(GestureType.SWIPE, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_B, gesture.keyCode)
        assertEquals(100f, gesture.startX)
        assertEquals(200f, gesture.startY)
        assertEquals(300f, gesture.endX)
        assertEquals(400f, gesture.endY)
        assertEquals(500L, gesture.duration)
    }
    
    @Test
    fun testMultiTapGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.MULTI_TAP,
            keyCode = KeyEvent.KEYCODE_C,
            startX = 100f,
            startY = 200f,
            tapCount = 3
        )
        
        assertEquals(GestureType.MULTI_TAP, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_C, gesture.keyCode)
        assertEquals(3, gesture.tapCount)
    }
    
    @Test
    fun testCircleGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.CIRCLE,
            keyCode = KeyEvent.KEYCODE_D,
            startX = 100f,
            startY = 200f,
            radius = 50f,
            duration = 1000L
        )
        
        assertEquals(GestureType.CIRCLE, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_D, gesture.keyCode)
        assertEquals(50f, gesture.radius)
        assertEquals(1000L, gesture.duration)
    }
}
Новый файл: CustomOverlayButtonTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito.*

class CustomOverlayButtonTest {
    
    private lateinit var context: Context
    private lateinit var button: CustomOverlayButton
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        button = CustomOverlayButton(context)




Продолжай

2. Добавление новых тестов (продолжение)
Продолжение файла: CustomOverlayButtonTest.kt
kotlin

Copy
@Test
    fun testDefaultValues() {
        assertEquals(120f, button.buttonSize)
        assertEquals(Color.parseColor("#80FFFFFF"), button.buttonColor)
        assertEquals(CustomOverlayButton.SHAPE_CIRCLE, button.buttonShape)
        assertEquals("", button.buttonText)
        assertEquals(Color.BLACK, button.buttonTextColor)
        assertEquals(Color.BLACK, button.buttonBorderColor)
        assertEquals(0f, button.buttonBorderWidth)
        assertEquals(128, button.buttonAlpha)
    }
    
    @Test
    fun testSetButtonProperties() {
        button.buttonSize = 200f
        button.buttonColor = Color.RED
        button.buttonShape = CustomOverlayButton.SHAPE_SQUARE
        button.buttonText = "Test"
        button.buttonTextColor = Color.BLUE
        button.buttonBorderColor = Color.GREEN
        button.buttonBorderWidth = 2f
        button.buttonAlpha = 200
        
        assertEquals(200f, button.buttonSize)
        assertEquals(Color.RED, button.buttonColor)
        assertEquals(CustomOverlayButton.SHAPE_SQUARE, button.buttonShape)
        assertEquals("Test", button.buttonText)
        assertEquals(Color.BLUE, button.buttonTextColor)
        assertEquals(Color.GREEN, button.buttonBorderColor)
        assertEquals(2f, button.buttonBorderWidth)
        assertEquals(200, button.buttonAlpha)
    }
    
    @Test
    fun testMeasureWithCustomSize() {
        button.buttonSize = 300f
        
        // Создаем макеты для тестирования
        val widthMeasureSpec = android.view.View.MeasureSpec.makeMeasureSpec(500, android.view.View.MeasureSpec.AT_MOST)
        val heightMeasureSpec = android.view.View.MeasureSpec.makeMeasureSpec(500, android.view.View.MeasureSpec.AT_MOST)
        
        button.measure(widthMeasureSpec, heightMeasureSpec)
        
        assertEquals(300, button.measuredWidth)
        assertEquals(300, button.measuredHeight)
    }
    
    @Test
    fun testDrawMethodCallsForDifferentShapes() {
        // Используем мок для Canvas
        val mockCanvas = mock(Canvas::class.java)
        
        // Тестируем круглую форму
        button.buttonShape = CustomOverlayButton.SHAPE_CIRCLE
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawCircle(anyFloat(), anyFloat(), anyFloat(), any())
        
        // Тестируем квадратную форму
        reset(mockCanvas)
        button.buttonShape = CustomOverlayButton.SHAPE_SQUARE
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawRect(any(), any())
        
        // Тестируем скругленную форму
        reset(mockCanvas)
        button.buttonShape = CustomOverlayButton.SHAPE_ROUNDED
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawRoundRect(any(), anyFloat(), anyFloat(), any())
    }
}
Новый файл: PersistenceManagerTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import java.io.File

class PersistenceManagerTest {
    
    private lateinit var context: Context
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        PersistenceManager.init(context)
        
        // Очищаем тестовые файлы перед каждым тестом
        val profilesFile = File(context.filesDir, "profiles.json")
        if (profilesFile.exists()) {
            profilesFile.delete()
        }
    }
    
    @Test
    fun testSaveAndLoadProfiles() {
        // Создаем тестовые профили
        val profile1 = GameProfile(name = "Test Profile 1")
        val profile2 = GameProfile(name = "Test Profile 2")
        
        // Сохраняем профили
        PersistenceManager.saveProfiles(listOf(profile1, profile2))
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что профили загружены корректно
        assertEquals(2, loadedProfiles.size)
        assertTrue(loadedProfiles.any { it.name == "Test Profile 1" })
        assertTrue(loadedProfiles.any { it.name == "Test Profile 2" })
    }
    
    @Test
    fun testSaveAndLoadEmptyProfiles() {
        // Сохраняем пустой список профилей
        PersistenceManager.saveProfiles(emptyList())
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что загружен пустой список
        assertTrue(loadedProfiles.isEmpty())
    }
    
    @Test
    fun testSaveAndLoadProfilesWithMappings() {
        // Создаем профиль с маппингами
        val profile = GameProfile(name = "Mapping Test")
        profile.keyMappings[android.view.KeyEvent.KEYCODE_A] = Pair(100f, 200f)
        profile.keyMappings[android.view.KeyEvent.KEYCODE_B] = Pair(300f, 400f)
        
        // Добавляем жест
        val gesture = GestureMapping(
            gestureType = GestureType.TAP,
            keyCode = android.view.KeyEvent.KEYCODE_X,
            startX = 150f,
            startY = 250f
        )
        profile.gestureMappings.add(gesture)
        
        // Сохраняем профиль
        PersistenceManager.saveProfiles(listOf(profile))
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что профиль загружен корректно
        assertEquals(1, loadedProfiles.size)
        val loadedProfile = loadedProfiles.first()
        
        // Проверяем маппинги
        assertEquals(2, loadedProfile.keyMappings.size)
        assertEquals(Pair(100f, 200f), loadedProfile.keyMappings[android.view.KeyEvent.KEYCODE_A])
        assertEquals(Pair(300f, 400f), loadedProfile.keyMappings[android.view.KeyEvent.KEYCODE_B])
        
        // Проверяем жесты
        assertEquals(1, loadedProfile.gestureMappings.size)
        val loadedGesture = loadedProfile.gestureMappings.first()
        assertEquals(GestureType.TAP, loadedGesture.gestureType)
        assertEquals(android.view.KeyEvent.KEYCODE_X, loadedGesture.keyCode)
        assertEquals(150f, loadedGesture.startX)
        assertEquals(250f, loadedGesture.startY)
    }
}
3. Применение принципов чистой архитектуры
Новый файл: repository/ProfileRepository.kt
kotlin

Copy
package com.example.gamemapper.repository

import com.example.gamemapper.GameProfile
import com.example.gamemapper.PersistenceManager

/**
 * Репозиторий для работы с профилями
 * Реализует паттерн Repository из чистой архитектуры
 */
interface ProfileRepository {
    fun getAllProfiles(): List<GameProfile>
    fun getProfileById(id: String): GameProfile?
    fun getProfileByPackage(packageName: String): GameProfile?
    fun saveProfile(profile: GameProfile)
    fun deleteProfile(id: String)
}

/**
 * Реализация репозитория профилей с использованием PersistenceManager
 */
class ProfileRepositoryImpl : ProfileRepository {
    private val profiles = mutableMapOf<String, GameProfile>()
    
    init {
        // Загружаем профили при инициализации
        loadProfiles()
    }
    
    /**
     * Загружает профили из хранилища
     */
    private fun loadProfiles() {
        val loadedProfiles = PersistenceManager.loadProfiles()
        profiles.clear()
        loadedProfiles.forEach { profiles[it.id] = it }
    }
    
    override fun getAllProfiles(): List<GameProfile> {
        return profiles.values.toList()
    }
    
    override fun getProfileById(id: String): GameProfile? {
        return profiles[id]
    }
    
    override fun getProfileByPackage(packageName: String): GameProfile? {
        return profiles.values.find { it.packageName == packageName }
    }
    
    override fun saveProfile(profile: GameProfile) {
        profiles[profile.id] = profile
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
    
    override fun deleteProfile(id: String) {
        profiles.remove(id)
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
}
Новый файл: repository/MappingRepository.kt
kotlin

Copy
package com.example.gamemapper.repository

import android.content.Context
import android.net.Uri
import com.example.gamemapper.RecorderManager

/**
 * Репозиторий для работы с маппингами
 * Реализует паттерн Repository из чистой архитектуры
 */
interface MappingRepository {
    fun saveMapping(context: Context, uri: Uri): Boolean
    fun loadMapping(context: Context, uri: Uri): Boolean
    fun getRecordedActions(): List<RecorderManager.TouchAction>
    fun clearRecordedActions()
}

/**
 * Реализация репозитория маппингов с использованием RecorderManager
 */
class MappingRepositoryImpl : MappingRepository {
    
    override fun saveMapping(context: Context, uri: Uri): Boolean {
        return RecorderManager.saveRecording(context, uri)
    }
    
    override fun loadMapping(context: Context, uri: Uri): Boolean {
        return RecorderManager.loadRecording(context, uri)
    }
    
    override fun getRecordedActions(): List<RecorderManager.TouchAction> {
        return RecorderManager.getRecordedActions()
    }
    
    override fun clearRecordedActions() {
        RecorderManager.clearPool()
    }
}
Новый файл: usecase/ProfileUseCase.kt
kotlin

Copy
package com.example.gamemapper.usecase

import com.example.gamemapper.GameProfile
import com.example.gamemapper.repository.ProfileRepository

/**
 * Use case для работы с профилями
 * Реализует бизнес-логику для работы с профилями
 */
class ProfileUseCase(private val repository: ProfileRepository) {
    
    /**
     * Получает все профили
     */
    fun getAllProfiles(): List<GameProfile> {
        return repository.getAllProfiles()
    }
    
    /**
     * Получает профиль по ID
     */
    fun getProfileById(id: String): GameProfile? {
        return repository.getProfileById(id)
    }
    
    /**
     * Получает профиль для указанного пакета приложения
     */
    fun getProfileForPackage(packageName: String): GameProfile? {
        return repository.getProfileByPackage(packageName)
    }
    
    /**
     * Создает новый профиль с указанным именем
     */
    fun createNewProfile(name: String): GameProfile {
        val profile = GameProfile(name = name)
        repository.saveProfile(profile)
        return profile
    }
    
    /**
     * Дублирует существующий профиль с новым именем
     */
    fun duplicateProfile(sourceId: String, newName: String): GameProfile? {
        val sourceProfile = repository.getProfileById(sourceId) ?: return null
        
        val newProfile = GameProfile(
            name = newName,
            packageName = sourceProfile.packageName
        )
        
        // Копируем настройки из исходного профиля
        newProfile.keyMappings.putAll(sourceProfile.keyMappings)
        newProfile.gestureMappings.addAll(sourceProfile.gestureMappings)
        newProfile.buttonSettings.putAll(sourceProfile.buttonSettings)
        
        repository.saveProfile(newProfile)
        return newProfile
    }
    
    /**
     * Сохраняет профиль
     */
    fun saveProfile(profile: GameProfile) {
        repository.saveProfile(profile)
    }
    
    /**
     * Удаляет профиль
     */
    fun deleteProfile(id: String) {
        repository.deleteProfile(id)
    }
}
Новый файл: usecase/MappingUseCase.kt
kotlin

Copy
package com.example.gamemapper.usecase

import android.content.Context
import android.net.Uri
import com.example.gamemapper.MappingService
import com.example.gamemapper.RecorderManager
import com.example.gamemapper.repository.MappingRepository

/**
 * Use case для работы с маппингами
 * Реализует бизнес-логику для работы с маппингами
 */
class MappingUseCase(private val repository: MappingRepository) {
    
    /**
     * Сохраняет маппинг в файл
     */
    fun saveMapping(context: Context, uri: Uri): Boolean {
        return repository.saveMapping(context, uri)
    }
    
    /**
     * Загружает маппинг из файла
     */
    fun loadMapping(context: Context, uri: Uri): Boolean {
        return repository.loadMapping(context, uri)
    }
    
    /**
     * Получает записанные действия
     */
    fun getRecordedActions(): List<RecorderManager.TouchAction> {
        return repository.getRecordedActions()
    }
    
    /**
     * Очищает записанные действия
     */
    fun clearRecordedActions() {
        repository.clearRecordedActions()
    }
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) {
        RecorderManager.playRecording(service, context)
    }
    
    /**
     * Начинает запись действий
     */
    fun startRecording() {
        RecorderManager.startRecording()
    }
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() {
        RecorderManager.stopRecording()
    }
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean {
        return RecorderManager.isRecording()
    }
}
Измененный файл: MainActivity.kt (частично)
kotlin

Copy
// Добавляем в начало файла
import com.example.gamemapper.repository.MappingRepositoryImpl
import com.example.gamemapper.repository.ProfileRepositoryImpl
import com.example.gamemapper.usecase.MappingUseCase
import com.example.gamemapper.usecase.ProfileUseCase

class MainActivity : AppCompatActivity()




Продолжай

Измененный файл: MainActivity.kt (продолжение)
kotlin

Copy
class MainActivity : AppCompatActivity() {
    private lateinit var startServiceButton: Button
    private lateinit var stopServiceButton: Button
    private lateinit var createMappingButton: Button
    private lateinit var editMappingButton: Button
    private lateinit var loadMappingButton: Button
    private lateinit var saveMappingButton: Button
    private lateinit var playButton: Button
    private lateinit var gamepadConfigButton: Button
    private lateinit var profilesButton: Button
    private lateinit var statusTextView: TextView
    private lateinit var recordIndicator: View
    
    // Добавляем Use Cases для чистой архитектуры
    private val profileRepository = ProfileRepositoryImpl()
    private val mappingRepository = MappingRepositoryImpl()
    private val profileUseCase = ProfileUseCase(profileRepository)
    private val mappingUseCase = MappingUseCase(mappingRepository)
    
    private val openDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.OpenDocument()
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_READ_URI_PERMISSION
            )
            if (mappingUseCase.loadMapping(this, it)) {
                Toast.makeText(this, R.string.mapping_loaded, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private val createDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.CreateDocument("text/plain")
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_WRITE_URI_PERMISSION
            )
            if (mappingUseCase.saveMapping(this, it)) {
                Toast.makeText(this, R.string.mapping_saved, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        startServiceButton = findViewById(R.id.startServiceButton)
        stopServiceButton = findViewById(R.id.stopServiceButton)
        createMappingButton = findViewById(R.id.createMappingButton)
        editMappingButton = findViewById(R.id.editMappingButton)
        loadMappingButton = findViewById(R.id.loadMappingButton)
        saveMappingButton = findViewById(R.id.saveMappingButton)
        playButton = findViewById(R.id.playButton)
        gamepadConfigButton = findViewById(R.id.gamepadConfigButton)
        profilesButton = findViewById(R.id.profilesButton)
        statusTextView = findViewById(R.id.statusTextView)
        recordIndicator = findViewById(R.id.recordIndicator)
        
        startServiceButton.setOnClickListener {
            if (!isAccessibilityServiceEnabled()) {
                promptEnableService()
            } else {
                startService(Intent(this, MappingService::class.java))
                Toast.makeText(this, R.string.service_started, Toast.LENGTH_SHORT).show()
                updateUI()
            }
        }
        
        stopServiceButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                stopService(Intent(this, MappingService::class.java).apply { action = "STOP_SERVICE" })
                updateUI()
            }
        }
        
        createMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                mappingUseCase.startRecording()
                recordIndicator.visibility = View.VISIBLE
                Toast.makeText(this, R.string.creating_mapping, Toast.LENGTH_SHORT).show()
            } else {
                promptEnableService()
            }
        }
        
        editMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                if (mappingUseCase.isRecording()) {
                    mappingUseCase.stopRecording()
                    recordIndicator.visibility = View.GONE
                }
                startActivity(Intent(this, EditMappingActivity::class.java))
            } else {
                promptEnableService()
            }
        }
        
        loadMappingButton.setOnClickListener {
            openDocumentLauncher.launch(arrayOf("text/plain"))
        }
        
        saveMappingButton.setOnClickListener {
            if (mappingUseCase.getRecordedActions().isEmpty()) {
                Toast.makeText(this, R.string.no_actions_to_save, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            createDocumentLauncher.launch("gamemapper_mapping_${System.currentTimeMillis()}.gmp")
        }
        
        playButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
                service?.let {
                     if (mappingUseCase.getRecordedActions().isEmpty()) {
                        Toast.makeText(this, R.string.no_actions_to_play, Toast.LENGTH_SHORT).show()
                    } else {
                        mappingUseCase.playRecording(it, this)
                     }
                }
            } else {
                promptEnableService()
            }
        }
        
        gamepadConfigButton.setOnClickListener {
            startActivity(Intent(this, GamepadConfigActivity::class.java))
        }
        
        profilesButton.setOnClickListener {
            startActivity(Intent(this, ProfileManagementActivity::class.java))
        }
        
        // Добавляем кнопку для настройки жестов
        val gestureConfigButton = findViewById<Button>(R.id.gestureConfigButton)
        gestureConfigButton.setOnClickListener {
            // Получаем все профили через useCase
            val profiles = profileUseCase.getAllProfiles()
            if (profiles.isEmpty()) {
                Toast.makeText(this, R.string.no_profiles_available, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Если есть только один профиль, открываем настройку жестов для него
            if (profiles.size == 1) {
                val intent = Intent(this, GestureConfigActivity::class.java).apply {
                    putExtra("profile_id", profiles[0].id)
                }
                startActivity(intent)
                return@setOnClickListener
            }
            
            // Если есть несколько профилей, показываем диалог выбора
            val profileNames = profiles.map { it.name }.toTypedArray()
            AlertDialog.Builder(this)
                .setTitle(R.string.select_profile_for_gestures)
                .setItems(profileNames) { _, which ->
                    val selectedProfile = profiles[which]
                    val intent = Intent(this, GestureConfigActivity::class.java).apply {
                        putExtra("profile_id", selectedProfile.id)
                    }
                    startActivity(intent)
                }
                .setNegativeButton(R.string.cancel, null)
                .show()
        }
        
        // Инициализируем PersistenceManager
        PersistenceManager.init(applicationContext)
        
        // Проверяем разрешения при первом запуске
        if (savedInstanceState == null) {
            checkPermissions()
        }
    }
    
    // Остальные методы класса остаются без изменений...
}
Измененный файл: ProfileManagementActivity.kt (частично)
kotlin

Copy
class ProfileManagementActivity : AppCompatActivity() {
    private lateinit var profilesRecyclerView: RecyclerView
    private lateinit var addProfileButton: Button
    private lateinit var adapter: ProfileAdapter
    
    // Добавляем Use Cases для чистой архитектуры
    private val profileRepository = ProfileRepositoryImpl()
    private val profileUseCase = ProfileUseCase(profileRepository)
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile_management)
        
        profilesRecyclerView = findViewById(R.id.profilesRecyclerView)
        addProfileButton = findViewById(R.id.addProfileButton)
        
        // Загружаем профили через Use Case
        val profiles = profileUseCase.getAllProfiles()
        
        adapter = ProfileAdapter(profiles.toMutableList(),
            onProfileSelected = { profile ->
                // Загружаем выбранный профиль в сервис
                val intent = Intent(this, MappingService::class.java).apply {
                    action = "LOAD_PROFILE"
                    putExtra("profile_id", profile.id)
                }
                startService(intent)
                Toast.makeText(this, getString(R.string.profile_activated, profile.name), Toast.LENGTH_SHORT).show()
            },
            onProfileEdit = { profile ->
                showEditProfileDialog(profile)
            },
            onProfileDelete = { profile ->
                showDeleteProfileDialog(profile)
            },
            onAssignApp = { profile ->
                showAppSelectionDialog(profile)
            }
        )
        
        profilesRecyclerView.adapter = adapter
        profilesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addProfileButton.setOnClickListener {
            showCreateProfileDialog()
        }
    }
    
    private fun showCreateProfileDialog() {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.create_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.create) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    // Создаем профиль через Use Case
                    val profile = profileUseCase.createNewProfile(name)
                    adapter.addProfile(profile)
                    Toast.makeText(this, getString(R.string.profile_created, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showEditProfileDialog(profile: GameProfile) {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        nameEditText.setText(profile.name)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.edit_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.save) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    profile.name = name
                    // Сохраняем профиль через Use Case
                    profileUseCase.saveProfile(profile)
                    adapter.notifyDataSetChanged()
                    Toast.makeText(this, getString(R.string.profile_updated, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showDeleteProfileDialog(profile: GameProfile) {
        AlertDialog.Builder(this)
            .setTitle(R.string.delete_profile)
            .setMessage(getString(R.string.delete_profile_confirmation, profile.name))
            .setPositiveButton(R.string.delete) { _, _ ->
                // Удаляем профиль через Use Case
                profileUseCase.deleteProfile(profile.id)
                adapter.removeProfile(profile)
                Toast.makeText(this, getString(R.string.profile_deleted, profile.name), Toast.LENGTH_SHORT).show()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    // Остальные методы класса остаются без изменений...
}
Измененный файл: GestureConfigActivity.kt (частично)
kotlin

Copy
class GestureConfigActivity : AppCompatActivity() {
    private lateinit var gesturesRecyclerView: RecyclerView
    private lateinit var addGestureButton: Button
    private lateinit var adapter: GestureAdapter
    private lateinit var currentProfile: GameProfile
    
    // Добавляем Use Cases для чистой архитектуры
    private val profileRepository = ProfileRepositoryImpl()
    private val profileUseCase = ProfileUseCase(profileRepository)
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_gesture_config)
        
        gesturesRecyclerView = findViewById(R.id.gesturesRecyclerView)
        addGestureButton = findViewById(R.id.addGestureButton)
        
        // Получаем текущий профиль через Use Case
        val profileId = intent.getStringExtra("profile_id") ?: return finish()
        currentProfile = profileUseCase.getProfileById(profileId) ?: return finish()
        
        title = getString(R.string.gesture_config_for, currentProfile.name)
        
        adapter = GestureAdapter(
            currentProfile.gestureMappings,
            onGestureEdit = { gesture ->
                showGestureDialog(gesture)
            },
            onGestureDelete = { gesture ->
                currentProfile.gestureMappings.remove(gesture)
                profileUseCase.saveProfile(currentProfile)
                adapter.notifyDataSetChanged()
            }
        )
        
        gesturesRecyclerView.adapter = adapter
        gesturesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addGestureButton.setOnClickListener {
            // Создаем новый жест по умолчанию
            val newGesture = GestureMapping(
                gestureType = GestureType.TAP,
                keyCode = android.view.KeyEvent.KEYCODE_UNKNOWN,
                startX = 500f,
                startY = 500f
            )
            showGestureDialog(newGesture, isNew = true)
        }
    }
    
    // Остальные методы класса остаются без изменений...
}
Новый файл: di/Modules.kt
kotlin

Copy
package com.example.gamemapper.di

import com.example.gamemapper.repository.MappingRepository
import com.example.gamemapper.repository.MappingRepositoryImpl
import com.example.gamemapper.repository.ProfileRepository
import com.example.gamemapper.repository.ProfileRepositoryImpl
import com.example.gamemapper.usecase.MappingUseCase
import com.example.gamemapper.usecase.ProfileUseCase

/**
 * Модуль для внедрения зависимостей (Dependency Injection)
 * Используется для создания и предоставления зависимостей в приложении
 */
object Modules {
    // Репозитории
    private val profileRepository: ProfileRepository by lazy { ProfileRepositoryImpl() }
    private val mappingRepository: MappingRepository by lazy { MappingRepositoryImpl() }
    
    // Use Cases
    val profileUseCase: ProfileUseCase by lazy { ProfileUseCase(profileRepository) }
    val mappingUseCase: MappingUseCase by lazy { MappingUse




Продолжай

Продолжение файла: di/Modules.kt
kotlin

Copy
// Use Cases
    val profileUseCase: ProfileUseCase by lazy { ProfileUseCase(profileRepository) }
    val mappingUseCase: MappingUseCase by lazy { MappingUseCase(mappingRepository) }
    
    /**
     * Инициализирует все необходимые компоненты приложения
     */
    fun initialize() {
        // Здесь можно добавить инициализацию других компонентов при необходимости
        // Например, загрузку профилей при старте приложения
        profileUseCase.getAllProfiles() // Это загрузит профили при первом обращении
    }
}
Новый файл: GameMapperApplication.kt
kotlin

Copy
package com.example.gamemapper

import android.app.Application
import com.example.gamemapper.di.Modules

/**
 * Класс приложения для инициализации глобальных компонентов
 */
class GameMapperApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Инициализируем PersistenceManager
        PersistenceManager.init(applicationContext)
        
        // Инициализируем модули DI
        Modules.initialize()
    }
}
Измененный файл: AndroidManifest.xml (добавление GameMapperApplication)
xml

Copy
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.gamemapper">
    
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    
    <application
        android:name=".GameMapperApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GameMapper">
        
        <!-- Остальное содержимое без изменений -->
        
    </application>
</manifest>
Обновление MainActivity.kt для использования Modules
kotlin

Copy
// Заменяем прямое создание репозиториев и use cases на использование Modules
class MainActivity : AppCompatActivity() {
    // Удаляем эти строки:
    // private val profileRepository = ProfileRepositoryImpl()
    // private val mappingRepository = MappingRepositoryImpl()
    // private val profileUseCase = ProfileUseCase(profileRepository)
    // private val mappingUseCase = MappingUseCase(mappingRepository)
    
    // Добавляем эти строки:
    private val profileUseCase = Modules.profileUseCase
    private val mappingUseCase = Modules.mappingUseCase
    
    // Остальной код без изменений
}
Обновление ProfileManagementActivity.kt для использования Modules
kotlin

Copy
class ProfileManagementActivity : AppCompatActivity() {
    // Удаляем эти строки:
    // private val profileRepository = ProfileRepositoryImpl()
    // private val profileUseCase = ProfileUseCase(profileRepository)
    
    // Добавляем эту строку:
    private val profileUseCase = Modules.profileUseCase
    
    // Остальной код без изменений
}
Обновление GestureConfigActivity.kt для использования Modules
kotlin

Copy
class GestureConfigActivity : AppCompatActivity() {
    // Удаляем эти строки:
    // private val profileRepository = ProfileRepositoryImpl()
    // private val profileUseCase = ProfileUseCase(profileRepository)
    
    // Добавляем эту строку:
    private val profileUseCase = Modules.profileUseCase
    
    // Остальной код без изменений
}
Новый файл: RecorderManagerImpl.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.util.Log
import android.widget.Toast
import java.io.BufferedReader
import java.io.InputStreamReader
import java.io.OutputStreamWriter
import java.util.concurrent.atomic.AtomicBoolean
import kotlinx.coroutines.*

/**
 * Реализация RecorderManager для записи и воспроизведения действий
 * Вынесена в отдельный класс для лучшей организации кода
 */
class RecorderManagerImpl {
    private val isRecording = AtomicBoolean(false)
    private val recordedActions = mutableListOf<RecorderManager.TouchAction>()
    
    /**
     * Начинает запись действий
     */
    fun startRecording() {
        if (!isRecording.getAndSet(true)) {
            recordedActions.clear()
            Log.d(TAG, "Recording started")
        }
    }
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() {
        if (isRecording.getAndSet(false)) {
            Log.d(TAG, "Recording stopped with ${recordedActions.size} actions")
        }
    }
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean = isRecording.get()
    
    /**
     * Записывает действие
     */
    fun recordAction(x: Float, y: Float, type: RecorderManager.ActionType) {
        if (isRecording.get()) {
            recordedActions.add(RecorderManager.TouchAction(x, y, type))
            Log.d(TAG, "Recorded action: $type at ($x, $y)")
        }
    }
    
    /**
     * Получает список записанных действий
     */
    fun getRecordedActions(): List<RecorderManager.TouchAction> = recordedActions.toList()
    
    /**
     * Очищает список записанных действий
     */
    fun clearPool() {
        recordedActions.clear()
        Log.d(TAG, "Action pool cleared")
    }
    
    /**
     * Сохраняет записанные действия в файл
     */
    fun saveRecording(context: Context, uri: Uri): Boolean {
        try {
            context.contentResolver.openOutputStream(uri)?.use { outputStream ->
                OutputStreamWriter(outputStream).use { writer ->
                    recordedActions.forEach { action ->
                        writer.write("${action.x},${action.y},${action.type},${action.timestamp}\n")
                    }
                }
            }
            Log.d(TAG, "Saved ${recordedActions.size} actions to $uri")
            return true
        } catch (e: Exception) {
            Log.e(TAG, "Error saving recording: ${e.message}", e)
            return false
        }
    }
    
    /**
     * Загружает записанные действия из файла
     */
    fun loadRecording(context: Context, uri: Uri): Boolean {
        try {
            val actions = mutableListOf<RecorderManager.TouchAction>()
            
            context.contentResolver.openInputStream(uri)?.use { inputStream ->
                BufferedReader(InputStreamReader(inputStream)).use { reader ->
                    reader.forEachLine { line ->
                        val parts = line.split(",")
                        if (parts.size >= 4) {
                            try {
                                val x = parts[0].toFloat()
                                val y = parts[1].toFloat()
                                val type = RecorderManager.ActionType.valueOf(parts[2])
                                val timestamp = parts[3].toLong()
                                actions.add(RecorderManager.TouchAction(x, y, type, timestamp))
                            } catch (e: Exception) {
                                Log.e(TAG, "Error parsing line: $line", e)
                            }
                        }
                    }
                }
            }
            
            if (actions.isNotEmpty()) {
                recordedActions.clear()
                recordedActions.addAll(actions)
                Log.d(TAG, "Loaded ${actions.size} actions from $uri")
                return true
            }
            return false
        } catch (e: Exception) {
            Log.e(TAG, "Error loading recording: ${e.message}", e)
            return false
        }
    }
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) {
        if (recordedActions.isEmpty()) {
            return
        }
        
        service.scope.launch {
            try {
                val startTime = System.currentTimeMillis()
                val firstActionTime = recordedActions.first().timestamp
                
                for (action in recordedActions) {
                    val elapsedTime = action.timestamp - firstActionTime
                    val waitTime = elapsedTime - (System.currentTimeMillis() - startTime)
                    
                    if (waitTime > 0) {
                        delay(waitTime)
                    }
                    
                    when (action.type) {
                        RecorderManager.ActionType.TAP -> {
                            service.performVirtualTouch(action.x, action.y)
                        }
                        RecorderManager.ActionType.SWIPE_START -> {
                            // Находим соответствующий SWIPE_END
                            val endAction = recordedActions.find {
                                 it.type == RecorderManager.ActionType.SWIPE_END && it.timestamp > action.timestamp
                             }
                            
                            if (endAction != null) {
                                service.simulateDrag(action.x, action.y, endAction.x, endAction.y)
                            }
                        }
                        RecorderManager.ActionType.LONG_PRESS -> {
                            service.performLongPress(action.x, action.y, 1000)
                        }
                        else -> {}
                    }
                }
                
                Toast.makeText(context, R.string.playback_completed, Toast.LENGTH_SHORT).show()
            } catch (e: Exception) {
                Log.e(TAG, "Error during playback: ${e.message}", e)
                Toast.makeText(context, R.string.playback_error, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    companion object {
        private const val TAG = "RecorderManagerImpl"
    }
}
Измененный файл: RecorderManager.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.util.Log

/**
 * Фасад для управления записью и воспроизведением действий
 * Реализует паттерн Singleton
 */
object RecorderManager {
    private val TAG = "RecorderManager"
    private val impl = RecorderManagerImpl()
    
    data class TouchAction(
        val x: Float,
        val y: Float,
        val type: ActionType,
        val timestamp: Long = System.currentTimeMillis()
    )
    
    enum class ActionType {
        TAP, SWIPE_START, SWIPE_END, LONG_PRESS
    }
    
    /**
     * Начинает запись действий
     */
    fun startRecording() = impl.startRecording()
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() = impl.stopRecording()
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean = impl.isRecording()
    
    /**
     * Записывает действие
     */
    fun recordAction(x: Float, y: Float, type: ActionType) = impl.recordAction(x, y, type)
    
    /**
     * Получает список записанных действий
     */
    fun getRecordedActions(): List<TouchAction> = impl.getRecordedActions()
    
    /**
     * Очищает список записанных действий
     */
    fun clearPool() = impl.clearPool()
    
    /**
     * Сохраняет записанные действия в файл
     */
    fun saveRecording(context: Context, uri: Uri): Boolean = impl.saveRecording(context, uri)
    
    /**
     * Загружает записанные действия из файла
     */
    fun loadRecording(context: Context, uri: Uri): Boolean = impl.loadRecording(context, uri)
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) = impl.playRecording(service, context)
}
Файлы, которые не требуют изменений:
ButtonSettingsDialog.kt
CustomOverlayButton.kt
EditMappingActivity.kt
GamepadConfigActivity.kt
KeyMappingAdapter.kt
MappingService.kt
Все XML-файлы макетов
Итоговые улучшения
В результате проведенных изменений мы:

Вынесли ProfileManager в отдельный файл для лучшей организации кода
Добавили новые тесты для повышения надежности приложения:
ProfileManagerTest.kt
GestureMappingTest.kt
CustomOverlayButtonTest.kt
PersistenceManagerTest.kt
Применили принципы чистой архитектуры:
Разделили код на слои (репозитории, use cases, UI)
Добавили интерфейсы для лучшей абстракции
Внедрили простую систему DI через объект Modules
Реализовали паттерн Repository
Реализовали паттерн Use Case
Добавили класс Application для инициализации глобальных компонентов
Эти изменения делают код более:

Модульным
Тестируемым
Поддерживаемым
Расширяемым
Кроме того, мы улучшили документацию кода, добавив комментарии KDoc ко всем новым и измененным классам и методам.





Покажи готовый вариант кода который ты поменял добавил или изменил полностью, с путями к файлам

Готовый вариант кода
Ниже представлены все файлы, которые были добавлены или изменены, с указанием полных путей.

Новые файлы
1. /app/src/main/java/com/example/gamemapper/ProfileManager.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.util.Log
import android.view.KeyEvent

/**
 * Менеджер профилей для управления игровыми профилями
 * Реализует паттерн Singleton для централизованного управления профилями
 */
object ProfileManager {
    private const val TAG = "ProfileManager"
    private val profiles = mutableMapOf<String, GameProfile>()
    
    /**
     * Инициализация менеджера профилей с профилем по умолчанию
     */
    init {
        // Создаем профиль по умолчанию
        val defaultProfile = GameProfile(
            name = "Default Profile"
        ).apply {
            keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
            keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
            keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
            keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
            keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
            keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
            keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
            keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
        }
        
        profiles[defaultProfile.id] = defaultProfile
    }
    
    /**
     * Сохраняет профиль в хранилище
     * @param profile Профиль для сохранения
     */
    fun saveProfile(profile: GameProfile) {
        profiles[profile.id] = profile
        PersistenceManager.saveProfiles(profiles.values.toList())
        Log.d(TAG, "Профиль сохранен: ${profile.name}")
    }
    
    /**
     * Получает профиль по идентификатору
     * @param id Идентификатор профиля
     * @return Профиль или null, если не найден
     */
    fun getProfile(id: String): GameProfile? {
        return profiles[id]
    }
    
    /**
     * Получает список всех профилей
     * @return Список всех профилей
     */
    fun getAllProfiles(): List<GameProfile> {
        return profiles.values.toList()
    }
    
    /**
     * Удаляет профиль по идентификатору
     * @param id Идентификатор профиля для удаления
     */
    fun deleteProfile(id: String) {
        val profile = profiles[id]
        profiles.remove(id)
        PersistenceManager.saveProfiles(profiles.values.toList())
        Log.d(TAG, "Профиль удален: ${profile?.name}")
    }
    
    /**
     * Получает профиль для указанного пакета приложения
     * @param packageName Имя пакета приложения
     * @return Профиль для указанного приложения или null
     */
    fun getProfileForPackage(packageName: String): GameProfile? {
        return profiles.values.find { it.packageName == packageName }
    }
    
    /**
     * Загружает профили из хранилища
     */
    fun loadProfiles() {
        val loadedProfiles = PersistenceManager.loadProfiles()
        profiles.clear()
        loadedProfiles.forEach { profiles[it.id] = it }
        
        // Если нет профилей, создаем профиль по умолчанию
        if (profiles.isEmpty()) {
            val defaultProfile = GameProfile(
                name = "Default Profile"
            ).apply {
                keyMappings[KeyEvent.KEYCODE_W] = Pair(500f, 300f)
                keyMappings[KeyEvent.KEYCODE_A] = Pair(200f, 500f)
                keyMappings[KeyEvent.KEYCODE_S] = Pair(500f, 700f)
                keyMappings[KeyEvent.KEYCODE_D] = Pair(800f, 500f)
                keyMappings[KeyEvent.KEYCODE_SPACE] = Pair(900f, 800f)
                keyMappings[KeyEvent.KEYCODE_E] = Pair(1000f, 300f)
                keyMappings[KeyEvent.KEYCODE_R] = Pair(1100f, 400f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_A] = Pair(900f, 700f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_B] = Pair(1000f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_X] = Pair(800f, 600f)
                keyMappings[KeyEvent.KEYCODE_BUTTON_Y] = Pair(900f, 500f)
            }
            
            profiles[defaultProfile.id] = defaultProfile
            PersistenceManager.saveProfiles(profiles.values.toList())
        }
        
        Log.d(TAG, "Загружено профилей: ${profiles.size}")
    }
    
    /**
     * Создает новый профиль с указанным именем
     * @param name Имя нового профиля
     * @return Созданный профиль
     */
    fun createNewProfile(name: String): GameProfile {
        val profile = GameProfile(name = name)
        saveProfile(profile)
        return profile
    }
    
    /**
     * Дублирует существующий профиль с новым именем
     * @param sourceId Идентификатор исходного профиля
     * @param newName Имя для нового профиля
     * @return Новый профиль или null если исходный профиль не найден
     */
    fun duplicateProfile(sourceId: String, newName: String): GameProfile? {
        val sourceProfile = getProfile(sourceId) ?: return null
        
        val newProfile = GameProfile(
            name = newName,
            packageName = sourceProfile.packageName
        )
        
        // Копируем настройки из исходного профиля
        newProfile.keyMappings.putAll(sourceProfile.keyMappings)
        newProfile.gestureMappings.addAll(sourceProfile.gestureMappings)
        newProfile.buttonSettings.putAll(sourceProfile.buttonSettings)
        
        saveProfile(newProfile)
        return newProfile
    }
}
2. /app/src/main/java/com/example/gamemapper/repository/ProfileRepository.kt
kotlin

Copy
package com.example.gamemapper.repository

import com.example.gamemapper.GameProfile
import com.example.gamemapper.PersistenceManager

/**
 * Репозиторий для работы с профилями
 * Реализует паттерн Repository из чистой архитектуры
 */
interface ProfileRepository {
    fun getAllProfiles(): List<GameProfile>
    fun getProfileById(id: String): GameProfile?
    fun getProfileByPackage(packageName: String): GameProfile?
    fun saveProfile(profile: GameProfile)
    fun deleteProfile(id: String)
}

/**
 * Реализация репозитория профилей с использованием PersistenceManager
 */
class ProfileRepositoryImpl : ProfileRepository {
    private val profiles = mutableMapOf<String, GameProfile>()
    
    init {
        // Загружаем профили при инициализации
        loadProfiles()
    }
    
    /**
     * Загружает профили из хранилища
     */
    private fun loadProfiles() {
        val loadedProfiles = PersistenceManager.loadProfiles()
        profiles.clear()
        loadedProfiles.forEach { profiles[it.id] = it }
    }
    
    override fun getAllProfiles(): List<GameProfile> {
        return profiles.values.toList()
    }
    
    override fun getProfileById(id: String): GameProfile? {
        return profiles[id]
    }
    
    override fun getProfileByPackage(packageName: String): GameProfile? {
        return profiles.values.find { it.packageName == packageName }
    }
    
    override fun saveProfile(profile: GameProfile) {
        profiles[profile.id] = profile
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
    
    override fun deleteProfile(id: String) {
        profiles.remove(id)
        PersistenceManager.saveProfiles(profiles.values.toList())
    }
}
3. /app/src/main/java/com/example/gamemapper/repository/MappingRepository.kt
kotlin

Copy
package com.example.gamemapper.repository

import android.content.Context
import android.net.Uri
import com.example.gamemapper.RecorderManager

/**
 * Репозиторий для работы с маппингами
 * Реализует паттерн Repository из чистой архитектуры
 */
interface MappingRepository {
    fun saveMapping(context: Context, uri: Uri): Boolean
    fun loadMapping(context: Context, uri: Uri): Boolean
    fun getRecordedActions(): List<RecorderManager.TouchAction>
    fun clearRecordedActions()
}

/**
 * Реализация репозитория маппингов с использованием RecorderManager
 */
class MappingRepositoryImpl : MappingRepository {
    
    override fun saveMapping(context: Context, uri: Uri): Boolean {
        return RecorderManager.saveRecording(context, uri)
    }
    
    override fun loadMapping(context: Context, uri: Uri): Boolean {
        return RecorderManager.loadRecording(context, uri)
    }
    
    override fun getRecordedActions(): List<RecorderManager.TouchAction> {
        return RecorderManager.getRecordedActions()
    }
    
    override fun clearRecordedActions() {
        RecorderManager.clearPool()
    }
}
4. /app/src/main/java/com/example/gamemapper/usecase/ProfileUseCase.kt
kotlin

Copy
package com.example.gamemapper.usecase

import com.example.gamemapper.GameProfile
import com.example.gamemapper.repository.ProfileRepository

/**
 * Use case для работы с профилями
 * Реализует бизнес-логику для работы с профилями
 */
class ProfileUseCase(private val repository: ProfileRepository) {
    
    /**
     * Получает все профили
     */
    fun getAllProfiles(): List<GameProfile> {
        return repository.getAllProfiles()
    }
    
    /**
     * Получает профиль по ID
     */
    fun getProfileById(id: String): GameProfile? {
        return repository.getProfileById(id)
    }
    
    /**
     * Получает профиль для указанного пакета приложения
     */
    fun getProfileForPackage(packageName: String): GameProfile? {
        return repository.getProfileByPackage(packageName)
    }
    
    /**
     * Создает новый профиль с указанным именем
     */
    fun createNewProfile(name: String): GameProfile {
        val profile = GameProfile(name = name)
        repository.saveProfile(profile)
        return profile
    }
    
    /**
     * Дублирует существующий профиль с новым именем
     */
    fun duplicateProfile(sourceId: String, newName: String): GameProfile? {
        val sourceProfile = repository.getProfileById(sourceId) ?: return null
        
        val newProfile = GameProfile(
            name = newName,
            packageName = sourceProfile.packageName
        )
        
        // Копируем настройки из исходного профиля
        newProfile.keyMappings.putAll(sourceProfile.keyMappings)
        newProfile.gestureMappings.addAll(sourceProfile.gestureMappings)
        newProfile.buttonSettings.putAll(sourceProfile.buttonSettings)
        
        repository.saveProfile(newProfile)
        return newProfile
    }
    
    /**
     * Сохраняет профиль
     */
    fun saveProfile(profile: GameProfile) {
        repository.saveProfile(profile)
    }
    
    /**
     * Удаляет профиль
     */
    fun deleteProfile(id: String) {
        repository.deleteProfile(id)
    }
}
5. /app/src/main/java/com/example/gamemapper/usecase/MappingUseCase.kt
kotlin

Copy
package com.example.gamemapper.usecase

import android.content.Context
import android.net.Uri
import com.example.gamemapper.MappingService
import com.example.gamemapper.RecorderManager
import com.example.gamemapper.repository.MappingRepository

/**
 * Use case для работы с маппингами
 * Реализует бизнес-логику для работы с маппингами
 */
class MappingUseCase(private val repository: MappingRepository) {
    
    /**
     * Сохраняет маппинг в файл
     */
    fun saveMapping(context: Context, uri: Uri): Boolean {
        return repository.saveMapping(context, uri)
    }
    
    /**
     * Загружает маппинг из файла
     */
    fun loadMapping(context: Context, uri: Uri): Boolean {
        return repository.loadMapping(context, uri)
    }
    
    /**
     * Получает записанные действия
     */
    fun getRecordedActions(): List<RecorderManager.TouchAction> {
        return repository.getRecordedActions()
    }
    
    /**
     * Очищает записанные действия
     */
    fun clearRecordedActions() {
        repository.clearRecordedActions()
    }
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) {
        RecorderManager.playRecording(service, context)
    }
    
    /**
     * Начинает запись действий
     */
    fun startRecording() {
        RecorderManager.startRecording()
    }
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() {
        RecorderManager.stopRecording()
    }
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean {
        return RecorderManager.isRecording()
    }
}
6. /app/src/main/java/com/example/gamemapper/di/Modules.kt
kotlin

Copy
package com.example.gamemapper.di

import com.example.gamemapper.repository.MappingRepository
import com.example.gamemapper.repository.MappingRepositoryImpl
import com.example.gamemapper.repository.ProfileRepository




Продолжай

6. /app/src/main/java/com/example/gamemapper/di/Modules.kt (продолжение)
kotlin

Copy
package com.example.gamemapper.di

import com.example.gamemapper.repository.MappingRepository
import com.example.gamemapper.repository.MappingRepositoryImpl
import com.example.gamemapper.repository.ProfileRepository
import com.example.gamemapper.repository.ProfileRepositoryImpl
import com.example.gamemapper.usecase.MappingUseCase
import com.example.gamemapper.usecase.ProfileUseCase

/**
 * Модуль для внедрения зависимостей (Dependency Injection)
 * Используется для создания и предоставления зависимостей в приложении
 */
object Modules {
    // Репозитории
    private val profileRepository: ProfileRepository by lazy { ProfileRepositoryImpl() }
    private val mappingRepository: MappingRepository by lazy { MappingRepositoryImpl() }
    
    // Use Cases
    val profileUseCase: ProfileUseCase by lazy { ProfileUseCase(profileRepository) }
    val mappingUseCase: MappingUseCase by lazy { MappingUseCase(mappingRepository) }
    
    /**
     * Инициализирует все необходимые компоненты приложения
     */
    fun initialize() {
        // Здесь можно добавить инициализацию других компонентов при необходимости
        // Например, загрузку профилей при старте приложения
        profileUseCase.getAllProfiles() // Это загрузит профили при первом обращении
    }
}
7. /app/src/main/java/com/example/gamemapper/GameMapperApplication.kt
kotlin

Copy
package com.example.gamemapper

import android.app.Application
import com.example.gamemapper.di.Modules

/**
 * Класс приложения для инициализации глобальных компонентов
 */
class GameMapperApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Инициализируем PersistenceManager
        PersistenceManager.init(applicationContext)
        
        // Инициализируем модули DI
        Modules.initialize()
    }
}
8. /app/src/main/java/com/example/gamemapper/RecorderManagerImpl.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.util.Log
import android.widget.Toast
import java.io.BufferedReader
import java.io.InputStreamReader
import java.io.OutputStreamWriter
import java.util.concurrent.atomic.AtomicBoolean
import kotlinx.coroutines.*

/**
 * Реализация RecorderManager для записи и воспроизведения действий
 * Вынесена в отдельный класс для лучшей организации кода
 */
class RecorderManagerImpl {
    private val isRecording = AtomicBoolean(false)
    private val recordedActions = mutableListOf<RecorderManager.TouchAction>()
    
    /**
     * Начинает запись действий
     */
    fun startRecording() {
        if (!isRecording.getAndSet(true)) {
            recordedActions.clear()
            Log.d(TAG, "Recording started")
        }
    }
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() {
        if (isRecording.getAndSet(false)) {
            Log.d(TAG, "Recording stopped with ${recordedActions.size} actions")
        }
    }
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean = isRecording.get()
    
    /**
     * Записывает действие
     */
    fun recordAction(x: Float, y: Float, type: RecorderManager.ActionType) {
        if (isRecording.get()) {
            recordedActions.add(RecorderManager.TouchAction(x, y, type))
            Log.d(TAG, "Recorded action: $type at ($x, $y)")
        }
    }
    
    /**
     * Получает список записанных действий
     */
    fun getRecordedActions(): List<RecorderManager.TouchAction> = recordedActions.toList()
    
    /**
     * Очищает список записанных действий
     */
    fun clearPool() {
        recordedActions.clear()
        Log.d(TAG, "Action pool cleared")
    }
    
    /**
     * Сохраняет записанные действия в файл
     */
    fun saveRecording(context: Context, uri: Uri): Boolean {
        try {
            context.contentResolver.openOutputStream(uri)?.use { outputStream ->
                OutputStreamWriter(outputStream).use { writer ->
                    recordedActions.forEach { action ->
                        writer.write("${action.x},${action.y},${action.type},${action.timestamp}\n")
                    }
                }
            }
            Log.d(TAG, "Saved ${recordedActions.size} actions to $uri")
            return true
        } catch (e: Exception) {
            Log.e(TAG, "Error saving recording: ${e.message}", e)
            return false
        }
    }
    
    /**
     * Загружает записанные действия из файла
     */
    fun loadRecording(context: Context, uri: Uri): Boolean {
        try {
            val actions = mutableListOf<RecorderManager.TouchAction>()
            
            context.contentResolver.openInputStream(uri)?.use { inputStream ->
                BufferedReader(InputStreamReader(inputStream)).use { reader ->
                    reader.forEachLine { line ->
                        val parts = line.split(",")
                        if (parts.size >= 4) {
                            try {
                                val x = parts[0].toFloat()
                                val y = parts[1].toFloat()
                                val type = RecorderManager.ActionType.valueOf(parts[2])
                                val timestamp = parts[3].toLong()
                                actions.add(RecorderManager.TouchAction(x, y, type, timestamp))
                            } catch (e: Exception) {
                                Log.e(TAG, "Error parsing line: $line", e)
                            }
                        }
                    }
                }
            }
            
            if (actions.isNotEmpty()) {
                recordedActions.clear()
                recordedActions.addAll(actions)
                Log.d(TAG, "Loaded ${actions.size} actions from $uri")
                return true
            }
            return false
        } catch (e: Exception) {
            Log.e(TAG, "Error loading recording: ${e.message}", e)
            return false
        }
    }
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) {
        if (recordedActions.isEmpty()) {
            return
        }
        
        service.scope.launch {
            try {
                val startTime = System.currentTimeMillis()
                val firstActionTime = recordedActions.first().timestamp
                
                for (action in recordedActions) {
                    val elapsedTime = action.timestamp - firstActionTime
                    val waitTime = elapsedTime - (System.currentTimeMillis() - startTime)
                    
                    if (waitTime > 0) {
                        delay(waitTime)
                    }
                    
                    when (action.type) {
                        RecorderManager.ActionType.TAP -> {
                            service.performVirtualTouch(action.x, action.y)
                        }
                        RecorderManager.ActionType.SWIPE_START -> {
                            // Находим соответствующий SWIPE_END
                            val endAction = recordedActions.find {
                                 it.type == RecorderManager.ActionType.SWIPE_END && it.timestamp > action.timestamp
                             }
                            
                            if (endAction != null) {
                                service.simulateDrag(action.x, action.y, endAction.x, endAction.y)
                            }
                        }
                        RecorderManager.ActionType.LONG_PRESS -> {
                            service.performLongPress(action.x, action.y, 1000)
                        }
                        else -> {}
                    }
                }
                
                Toast.makeText(context, R.string.playback_completed, Toast.LENGTH_SHORT).show()
            } catch (e: Exception) {
                Log.e(TAG, "Error during playback: ${e.message}", e)
                Toast.makeText(context, R.string.playback_error, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    companion object {
        private const val TAG = "RecorderManagerImpl"
    }
}
9. /app/src/test/java/com/example/gamemapper/ProfileManagerTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.view.KeyEvent
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito.*

class ProfileManagerTest {
    
    private lateinit var context: Context
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        // Мокируем PersistenceManager для тестирования
        val mockPersistenceManager = mock(PersistenceManager::class.java)
        `when`(mockPersistenceManager.loadProfiles()).thenReturn(emptyList())
    }
    
    @Test
    fun testCreateNewProfile() {
        val profileName = "Test Profile"
        val profile = ProfileManager.createNewProfile(profileName)
        
        assertEquals(profileName, profile.name)
        assertTrue(ProfileManager.getAllProfiles().contains(profile))
    }
    
    @Test
    fun testGetProfileForPackage() {
        val packageName = "com.example.testapp"
        val profile = ProfileManager.createNewProfile("Package Test")
        profile.packageName = packageName
        ProfileManager.saveProfile(profile)
        
        val retrievedProfile = ProfileManager.getProfileForPackage(packageName)
        assertNotNull(retrievedProfile)
        assertEquals(profile.id, retrievedProfile?.id)
    }
    
    @Test
    fun testDeleteProfile() {
        val profile = ProfileManager.createNewProfile("Delete Test")
        val profileId = profile.id
        
        assertTrue(ProfileManager.getAllProfiles().any { it.id == profileId })
        
        ProfileManager.deleteProfile(profileId)
        
        assertFalse(ProfileManager.getAllProfiles().any { it.id == profileId })
        assertNull(ProfileManager.getProfile(profileId))
    }
    
    @Test
    fun testDuplicateProfile() {
        val originalName = "Original Profile"
        val duplicateName = "Duplicate Profile"
        
        val originalProfile = ProfileManager.createNewProfile(originalName)
        originalProfile.keyMappings[KeyEvent.KEYCODE_A] = Pair(100f, 200f)
        originalProfile.keyMappings[KeyEvent.KEYCODE_B] = Pair(300f, 400f)
        ProfileManager.saveProfile(originalProfile)
        
        val duplicatedProfile = ProfileManager.duplicateProfile(originalProfile.id, duplicateName)
        
        assertNotNull(duplicatedProfile)
        assertEquals(duplicateName, duplicatedProfile?.name)
        assertEquals(originalProfile.keyMappings.size, duplicatedProfile?.keyMappings?.size)
        assertEquals(
            originalProfile.keyMappings[KeyEvent.KEYCODE_A], 
            duplicatedProfile?.keyMappings?.get(KeyEvent.KEYCODE_A)
        )
    }
}
10. /app/src/test/java/com/example/gamemapper/GestureMappingTest.kt
kotlin

Copy
package com.example.gamemapper

import android.view.KeyEvent
import org.junit.Assert.*
import org.junit.Test

class GestureMappingTest {
    
    @Test
    fun testTapGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.TAP,
            keyCode = KeyEvent.KEYCODE_A,
            startX = 100f,
            startY = 200f
        )
        
        assertEquals(GestureType.TAP, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_A, gesture.keyCode)
        assertEquals(100f, gesture.startX)
        assertEquals(200f, gesture.startY)
        assertNull(gesture.endX)
        assertNull(gesture.endY)
        assertEquals(0L, gesture.duration)
        assertEquals(1, gesture.tapCount)
        assertEquals(0f, gesture.radius)
    }
    
    @Test
    fun testSwipeGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.SWIPE,
            keyCode = KeyEvent.KEYCODE_B,
            startX = 100f,
            startY = 200f,
            endX = 300f,
            endY = 400f,
            duration = 500L
        )
        
        assertEquals(GestureType.SWIPE, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_B, gesture.keyCode)
        assertEquals(100f, gesture.startX)
        assertEquals(200f, gesture.startY)
        assertEquals(300f, gesture.endX)
        assertEquals(400f, gesture.endY)
        assertEquals(500L, gesture.duration)
    }
    
    @Test
    fun testMultiTapGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.MULTI_TAP,
            keyCode = KeyEvent.KEYCODE_C,
            startX = 100f,
            startY = 200f,
            tapCount = 3
        )
        
        assertEquals(GestureType.MULTI_TAP, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_C, gesture.keyCode)
        assertEquals(3, gesture.tapCount)
    }
    
    @Test
    fun testCircleGestureCreation() {
        val gesture = GestureMapping(
            gestureType = GestureType.CIRCLE,
            keyCode = KeyEvent.KEYCODE_D,
            startX = 100f,
            startY = 200f,
            radius = 50f,
            duration = 1000L
        )
        
        assertEquals(GestureType.CIRCLE, gesture.gestureType)
        assertEquals(KeyEvent.KEYCODE_D, gesture.keyCode)
        assertEquals(50f, gesture.radius)
        assertEquals(1000L, gesture.duration)
    }
}
11. /app/src/test/java/com/example/gamemapper/CustomOverlayButtonTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import org.mockito.Mockito.*

class CustomOverlayButtonTest {
    
    private lateinit var context: Context
    private lateinit var button: CustomOverlayButton
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        button = CustomOverlayButton(context)
    }
    
    @Test
    fun testDefaultValues() {
        assertEquals(120f, button.buttonSize)
        assertEquals(Color.parseColor("#80FFFFFF"), button.buttonColor)
        assertEquals(CustomOverlayButton.SHAPE_CIRCLE, button.buttonShape)
        assertEquals("", button.




Продолжай

11. /app/src/test/java/com/example/gamemapper/CustomOverlayButtonTest.kt (продолжение)
kotlin

Copy
assertEquals("", button.buttonText)
        assertEquals(Color.BLACK, button.buttonTextColor)
        assertEquals(Color.BLACK, button.buttonBorderColor)
        assertEquals(0f, button.buttonBorderWidth)
        assertEquals(128, button.buttonAlpha)
    }
    
    @Test
    fun testSetButtonProperties() {
        button.buttonSize = 200f
        button.buttonColor = Color.RED
        button.buttonShape = CustomOverlayButton.SHAPE_SQUARE
        button.buttonText = "Test"
        button.buttonTextColor = Color.BLUE
        button.buttonBorderColor = Color.GREEN
        button.buttonBorderWidth = 2f
        button.buttonAlpha = 200
        
        assertEquals(200f, button.buttonSize)
        assertEquals(Color.RED, button.buttonColor)
        assertEquals(CustomOverlayButton.SHAPE_SQUARE, button.buttonShape)
        assertEquals("Test", button.buttonText)
        assertEquals(Color.BLUE, button.buttonTextColor)
        assertEquals(Color.GREEN, button.buttonBorderColor)
        assertEquals(2f, button.buttonBorderWidth)
        assertEquals(200, button.buttonAlpha)
    }
    
    @Test
    fun testMeasureWithCustomSize() {
        button.buttonSize = 300f
        
        // Создаем макеты для тестирования
        val widthMeasureSpec = android.view.View.MeasureSpec.makeMeasureSpec(500, android.view.View.MeasureSpec.AT_MOST)
        val heightMeasureSpec = android.view.View.MeasureSpec.makeMeasureSpec(500, android.view.View.MeasureSpec.AT_MOST)
        
        button.measure(widthMeasureSpec, heightMeasureSpec)
        
        assertEquals(300, button.measuredWidth)
        assertEquals(300, button.measuredHeight)
    }
    
    @Test
    fun testDrawMethodCallsForDifferentShapes() {
        // Используем мок для Canvas
        val mockCanvas = mock(Canvas::class.java)
        
        // Тестируем круглую форму
        button.buttonShape = CustomOverlayButton.SHAPE_CIRCLE
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawCircle(anyFloat(), anyFloat(), anyFloat(), any())
        
        // Тестируем квадратную форму
        reset(mockCanvas)
        button.buttonShape = CustomOverlayButton.SHAPE_SQUARE
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawRect(any(), any())
        
        // Тестируем скругленную форму
        reset(mockCanvas)
        button.buttonShape = CustomOverlayButton.SHAPE_ROUNDED
        button.draw(mockCanvas)
        verify(mockCanvas, atLeastOnce()).drawRoundRect(any(), anyFloat(), anyFloat(), any())
    }
}
12. /app/src/test/java/com/example/gamemapper/PersistenceManagerTest.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import androidx.test.core.app.ApplicationProvider
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import java.io.File

class PersistenceManagerTest {
    
    private lateinit var context: Context
    
    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        PersistenceManager.init(context)
        
        // Очищаем тестовые файлы перед каждым тестом
        val profilesFile = File(context.filesDir, "profiles.json")
        if (profilesFile.exists()) {
            profilesFile.delete()
        }
    }
    
    @Test
    fun testSaveAndLoadProfiles() {
        // Создаем тестовые профили
        val profile1 = GameProfile(name = "Test Profile 1")
        val profile2 = GameProfile(name = "Test Profile 2")
        
        // Сохраняем профили
        PersistenceManager.saveProfiles(listOf(profile1, profile2))
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что профили загружены корректно
        assertEquals(2, loadedProfiles.size)
        assertTrue(loadedProfiles.any { it.name == "Test Profile 1" })
        assertTrue(loadedProfiles.any { it.name == "Test Profile 2" })
    }
    
    @Test
    fun testSaveAndLoadEmptyProfiles() {
        // Сохраняем пустой список профилей
        PersistenceManager.saveProfiles(emptyList())
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что загружен пустой список
        assertTrue(loadedProfiles.isEmpty())
    }
    
    @Test
    fun testSaveAndLoadProfilesWithMappings() {
        // Создаем профиль с маппингами
        val profile = GameProfile(name = "Mapping Test")
        profile.keyMappings[android.view.KeyEvent.KEYCODE_A] = Pair(100f, 200f)
        profile.keyMappings[android.view.KeyEvent.KEYCODE_B] = Pair(300f, 400f)
        
        // Добавляем жест
        val gesture = GestureMapping(
            gestureType = GestureType.TAP,
            keyCode = android.view.KeyEvent.KEYCODE_X,
            startX = 150f,
            startY = 250f
        )
        profile.gestureMappings.add(gesture)
        
        // Сохраняем профиль
        PersistenceManager.saveProfiles(listOf(profile))
        
        // Загружаем профили
        val loadedProfiles = PersistenceManager.loadProfiles()
        
        // Проверяем, что профиль загружен корректно
        assertEquals(1, loadedProfiles.size)
        val loadedProfile = loadedProfiles.first()
        
        // Проверяем маппинги
        assertEquals(2, loadedProfile.keyMappings.size)
        assertEquals(Pair(100f, 200f), loadedProfile.keyMappings[android.view.KeyEvent.KEYCODE_A])
        assertEquals(Pair(300f, 400f), loadedProfile.keyMappings[android.view.KeyEvent.KEYCODE_B])
        
        // Проверяем жесты
        assertEquals(1, loadedProfile.gestureMappings.size)
        val loadedGesture = loadedProfile.gestureMappings.first()
        assertEquals(GestureType.TAP, loadedGesture.gestureType)
        assertEquals(android.view.KeyEvent.KEYCODE_X, loadedGesture.keyCode)
        assertEquals(150f, loadedGesture.startX)
        assertEquals(250f, loadedGesture.startY)
    }
}
Измененные файлы
1. /app/src/main/java/com/example/gamemapper/GameProfile.kt
kotlin

Copy
package com.example.gamemapper

import android.view.KeyEvent
import java.util.UUID

/**
 * Модель данных для игрового профиля
 */
data class GameProfile(
    val id: String = UUID.randomUUID().toString(),
    var name: String,
    var packageName: String = "",
    val keyMappings: MutableMap<Int, Pair<Float, Float>> = mutableMapOf(),
    val gestureMappings: MutableList<GestureMapping> = mutableListOf(),
    val buttonSettings: MutableMap<Int, ButtonSettings> = mutableMapOf()
)

/**
 * Настройки кнопки для профиля
 */
data class ButtonSettings(
    val shape: Int = CustomOverlayButton.SHAPE_CIRCLE,
    val color: Int = android.graphics.Color.parseColor("#80FFFFFF"),
    val size: Float = 120f,
    val alpha: Int = 128,
    val borderColor: Int = android.graphics.Color.BLACK,
    val borderWidth: Float = 0f
)

/**
 * Модель данных для маппинга жестов
 */
data class GestureMapping(
    val id: String = UUID.randomUUID().toString(),
    val gestureType: GestureType,
    val keyCode: Int,
    val startX: Float,
    val startY: Float,
    val endX: Float? = null,
    val endY: Float? = null,
    val duration: Long = 0,
    val tapCount: Int = 1,
    val radius: Float = 0f
)

/**
 * Типы поддерживаемых жестов
 */
enum class GestureType {
    TAP, LONG_PRESS, SWIPE, MULTI_TAP, CIRCLE
}
2. /app/src/main/java/com/example/gamemapper/RecorderManager.kt
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.net.Uri
import android.util.Log

/**
 * Фасад для управления записью и воспроизведением действий
 * Реализует паттерн Singleton
 */
object RecorderManager {
    private val TAG = "RecorderManager"
    private val impl = RecorderManagerImpl()
    
    data class TouchAction(
        val x: Float,
        val y: Float,
        val type: ActionType,
        val timestamp: Long = System.currentTimeMillis()
    )
    
    enum class ActionType {
        TAP, SWIPE_START, SWIPE_END, LONG_PRESS
    }
    
    /**
     * Начинает запись действий
     */
    fun startRecording() = impl.startRecording()
    
    /**
     * Останавливает запись действий
     */
    fun stopRecording() = impl.stopRecording()
    
    /**
     * Проверяет, ведется ли запись
     */
    fun isRecording(): Boolean = impl.isRecording()
    
    /**
     * Записывает действие
     */
    fun recordAction(x: Float, y: Float, type: ActionType) = impl.recordAction(x, y, type)
    
    /**
     * Получает список записанных действий
     */
    fun getRecordedActions(): List<TouchAction> = impl.getRecordedActions()
    
    /**
     * Очищает список записанных действий
     */
    fun clearPool() = impl.clearPool()
    
    /**
     * Сохраняет записанные действия в файл
     */
    fun saveRecording(context: Context, uri: Uri): Boolean = impl.saveRecording(context, uri)
    
    /**
     * Загружает записанные действия из файла
     */
    fun loadRecording(context: Context, uri: Uri): Boolean = impl.loadRecording(context, uri)
    
    /**
     * Воспроизводит записанные действия
     */
    fun playRecording(service: MappingService, context: Context) = impl.playRecording(service, context)
}
3. /app/src/main/java/com/example/gamemapper/MainActivity.kt
kotlin

Copy
package com.example.gamemapper

import android.app.AlertDialog
import android.content.Context
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.provider.Settings
import android.view.View
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import com.example.gamemapper.di.Modules

class MainActivity : AppCompatActivity() {
    private lateinit var startServiceButton: Button
    private lateinit var stopServiceButton: Button
    private lateinit var createMappingButton: Button
    private lateinit var editMappingButton: Button
    private lateinit var loadMappingButton: Button
    private lateinit var saveMappingButton: Button
    private lateinit var playButton: Button
    private lateinit var gamepadConfigButton: Button
    private lateinit var profilesButton: Button
    private lateinit var statusTextView: TextView
    private lateinit var recordIndicator: View
    
    // Используем модули DI
    private val profileUseCase = Modules.profileUseCase
    private val mappingUseCase = Modules.mappingUseCase
    
    private val openDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.OpenDocument()
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_READ_URI_PERMISSION
            )
            if (mappingUseCase.loadMapping(this, it)) {
                Toast.makeText(this, R.string.mapping_loaded, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private val createDocumentLauncher = registerForActivityResult(
        ActivityResultContracts.CreateDocument("text/plain")
    ) { uri ->
        uri?.let {
            contentResolver.takePersistableUriPermission(
                it,
                Intent.FLAG_GRANT_WRITE_URI_PERMISSION
            )
            if (mappingUseCase.saveMapping(this, it)) {
                Toast.makeText(this, R.string.mapping_saved, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        startServiceButton = findViewById(R.id.startServiceButton)
        stopServiceButton = findViewById(R.id.stopServiceButton)
        createMappingButton = findViewById(R.id.createMappingButton)
        editMappingButton = findViewById(R.id.editMappingButton)
        loadMappingButton = findViewById(R.id.loadMappingButton)
        saveMappingButton = findViewById(R.id.saveMappingButton)
        playButton = findViewById(R.id.playButton)
        gamepadConfigButton = findViewById(R.id.gamepadConfigButton)
        profilesButton = findViewById(R.id.profilesButton)
        statusTextView = findViewById(R.id.statusTextView)
        recordIndicator = findViewById(R.id.recordIndicator)
        
        startServiceButton.setOnClickListener {
            if (!isAccessibilityServiceEnabled()) {
                promptEnableService()
            } else {
                startService(Intent(this, MappingService::class.java))
                Toast.makeText(this, R.string.service_started, Toast.LENGTH_SHORT).show()
                updateUI()
            }
        }
        
        stopServiceButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                stopService(Intent(this, MappingService::class.java).apply { action = "STOP_SERVICE" })
                updateUI()
            }
        }
        
        createMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                mappingUseCase.startRecording()
                recordIndicator.visibility = View.VISIBLE
                Toast.makeText(this, R.string.creating_mapping, Toast.LENGTH_SHORT).show()
            } else {
                promptEnableService()
            }
        }
        
        editMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {




Продолжай

3. /app/src/main/java/com/example/gamemapper/MainActivity.kt (продолжение)
kotlin

Copy
editMappingButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                if (mappingUseCase.isRecording()) {
                    mappingUseCase.stopRecording()
                    recordIndicator.visibility = View.GONE
                }
                startActivity(Intent(this, EditMappingActivity::class.java))
            } else {
                promptEnableService()
            }
        }
        
        loadMappingButton.setOnClickListener {
            openDocumentLauncher.launch(arrayOf("text/plain"))
        }
        
        saveMappingButton.setOnClickListener {
            if (mappingUseCase.getRecordedActions().isEmpty()) {
                Toast.makeText(this, R.string.no_actions_to_save, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            createDocumentLauncher.launch("gamemapper_mapping_${System.currentTimeMillis()}.gmp")
        }
        
        playButton.setOnClickListener {
            if (isAccessibilityServiceEnabled()) {
                val service = getSystemService(Context.ACCESSIBILITY_SERVICE) as? MappingService
                service?.let {
                     if (mappingUseCase.getRecordedActions().isEmpty()) {
                        Toast.makeText(this, R.string.no_actions_to_play, Toast.LENGTH_SHORT).show()
                    } else {
                        mappingUseCase.playRecording(it, this)
                     }
                }
            } else {
                promptEnableService()
            }
        }
        
        gamepadConfigButton.setOnClickListener {
            startActivity(Intent(this, GamepadConfigActivity::class.java))
        }
        
        profilesButton.setOnClickListener {
            startActivity(Intent(this, ProfileManagementActivity::class.java))
        }
        
        // Добавляем кнопку для настройки жестов
        val gestureConfigButton = findViewById<Button>(R.id.gestureConfigButton)
        gestureConfigButton.setOnClickListener {
            // Получаем все профили через useCase
            val profiles = profileUseCase.getAllProfiles()
            if (profiles.isEmpty()) {
                Toast.makeText(this, R.string.no_profiles_available, Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Если есть только один профиль, открываем настройку жестов для него
            if (profiles.size == 1) {
                val intent = Intent(this, GestureConfigActivity::class.java).apply {
                    putExtra("profile_id", profiles[0].id)
                }
                startActivity(intent)
                return@setOnClickListener
            }
            
            // Если есть несколько профилей, показываем диалог выбора
            val profileNames = profiles.map { it.name }.toTypedArray()
            AlertDialog.Builder(this)
                .setTitle(R.string.select_profile_for_gestures)
                .setItems(profileNames) { _, which ->
                    val selectedProfile = profiles[which]
                    val intent = Intent(this, GestureConfigActivity::class.java).apply {
                        putExtra("profile_id", selectedProfile.id)
                    }
                    startActivity(intent)
                }
                .setNegativeButton(R.string.cancel, null)
                .show()
        }
        
        // Проверяем разрешения при первом запуске
        if (savedInstanceState == null) {
            checkPermissions()
        }
    }
    
    override fun onResume() {
        super.onResume()
        updateUI()
    }
    
    private fun updateUI() {
        val serviceEnabled = isAccessibilityServiceEnabled()
        stopServiceButton.isEnabled = serviceEnabled
        createMappingButton.isEnabled = serviceEnabled
        editMappingButton.isEnabled = serviceEnabled
        loadMappingButton.isEnabled = true
        saveMappingButton.isEnabled = mappingUseCase.getRecordedActions().isNotEmpty()
        playButton.isEnabled = serviceEnabled && mappingUseCase.getRecordedActions().isNotEmpty()
        gamepadConfigButton.isEnabled = serviceEnabled
        profilesButton.isEnabled = true
        findViewById<Button>(R.id.gestureConfigButton).isEnabled = profileUseCase.getAllProfiles().isNotEmpty()
        
        statusTextView.text = getString(
            if (serviceEnabled) R.string.service_running else R.string.service_not_running
        ).let { "${getString(R.string.status_label)} $it" }
        
        recordIndicator.visibility = if (mappingUseCase.isRecording()) View.VISIBLE else View.GONE
    }
    
    private fun isAccessibilityServiceEnabled(): Boolean {
        val service = "$packageName/.MappingService"
        val enabledServices = Settings.Secure.getString(
            contentResolver,
            Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES
        )
        return enabledServices?.contains(service) == true
    }
    
    private fun promptEnableService() {
        AlertDialog.Builder(this)
            .setTitle(R.string.service_required)
            .setMessage(R.string.service_explanation)
            .setPositiveButton(R.string.go_to_settings) { _, _ ->
                startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun checkPermissions() {
        if (!Settings.canDrawOverlays(this)) {
            AlertDialog.Builder(this)
                .setTitle(R.string.overlay_permission_required)
                .setMessage(R.string.overlay_permission_explanation)
                .setPositiveButton(R.string.go_to_settings) { _, _ ->
                    val intent = Intent(
                        Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                        Uri.parse("package:$packageName")
                    )
                    startActivity(intent)
                }
                .setNegativeButton(R.string.cancel, null)
                .show()
        }
    }
}
4. /app/src/main/java/com/example/gamemapper/ProfileManagementActivity.kt
kotlin

Copy
package com.example.gamemapper

import android.app.AlertDialog
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.EditText
import android.widget.ImageButton
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.gamemapper.di.Modules

class ProfileManagementActivity : AppCompatActivity() {
    private lateinit var profilesRecyclerView: RecyclerView
    private lateinit var addProfileButton: Button
    private lateinit var adapter: ProfileAdapter
    
    // Используем модули DI
    private val profileUseCase = Modules.profileUseCase
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile_management)
        
        profilesRecyclerView = findViewById(R.id.profilesRecyclerView)
        addProfileButton = findViewById(R.id.addProfileButton)
        
        // Загружаем профили через Use Case
        val profiles = profileUseCase.getAllProfiles()
        
        adapter = ProfileAdapter(profiles.toMutableList(),
            onProfileSelected = { profile ->
                // Загружаем выбранный профиль в сервис
                val intent = Intent(this, MappingService::class.java).apply {
                    action = "LOAD_PROFILE"
                    putExtra("profile_id", profile.id)
                }
                startService(intent)
                Toast.makeText(this, getString(R.string.profile_activated, profile.name), Toast.LENGTH_SHORT).show()
            },
            onProfileEdit = { profile ->
                showEditProfileDialog(profile)
            },
            onProfileDelete = { profile ->
                showDeleteProfileDialog(profile)
            },
            onAssignApp = { profile ->
                showAppSelectionDialog(profile)
            }
        )
        
        profilesRecyclerView.adapter = adapter
        profilesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addProfileButton.setOnClickListener {
            showCreateProfileDialog()
        }
    }
    
    private fun showCreateProfileDialog() {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.create_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.create) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    // Создаем профиль через Use Case
                    val profile = profileUseCase.createNewProfile(name)
                    adapter.addProfile(profile)
                    Toast.makeText(this, getString(R.string.profile_created, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showEditProfileDialog(profile: GameProfile) {
        val dialogView = layoutInflater.inflate(R.layout.dialog_create_profile, null)
        val nameEditText = dialogView.findViewById<EditText>(R.id.profileNameEditText)
        nameEditText.setText(profile.name)
        
        AlertDialog.Builder(this)
            .setTitle(R.string.edit_profile)
            .setView(dialogView)
            .setPositiveButton(R.string.save) { _, _ ->
                val name = nameEditText.text.toString().trim()
                if (name.isNotEmpty()) {
                    profile.name = name
                    // Сохраняем профиль через Use Case
                    profileUseCase.saveProfile(profile)
                    adapter.notifyDataSetChanged()
                    Toast.makeText(this, getString(R.string.profile_updated, name), Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, R.string.profile_name_required, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showDeleteProfileDialog(profile: GameProfile) {
        AlertDialog.Builder(this)
            .setTitle(R.string.delete_profile)
            .setMessage(getString(R.string.delete_profile_confirmation, profile.name))
            .setPositiveButton(R.string.delete) { _, _ ->
                // Удаляем профиль через Use Case
                profileUseCase.deleteProfile(profile.id)
                adapter.removeProfile(profile)
                Toast.makeText(this, getString(R.string.profile_deleted, profile.name), Toast.LENGTH_SHORT).show()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun showAppSelectionDialog(profile: GameProfile) {
        val pm = packageManager
        val installedApps = pm.getInstalledApplications(PackageManager.GET_META_DATA)
            .filter { pm.getLaunchIntentForPackage(it.packageName) != null }
            .sortedBy { pm.getApplicationLabel(it).toString() }
        
        val appNames = installedApps.map { pm.getApplicationLabel(it).toString() }.toTypedArray()
        
        AlertDialog.Builder(this)
            .setTitle(R.string.select_app)
            .setItems(appNames) { _, which ->
                val selectedApp = installedApps[which]
                profile.packageName = selectedApp.packageName
                
                // Сохраняем профиль через Use Case
                profileUseCase.saveProfile(profile)
                adapter.notifyDataSetChanged()
                
                Toast.makeText(
                    this, 
                    getString(R.string.app_assigned, appNames[which], profile.name),
                    Toast.LENGTH_SHORT
                ).show()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    // Адаптер для списка профилей
    class ProfileAdapter(
        private val profiles: MutableList<GameProfile>,
        private val onProfileSelected: (GameProfile) -> Unit,
        private val onProfileEdit: (GameProfile) -> Unit,
        private val onProfileDelete: (GameProfile) -> Unit,
        private val onAssignApp: (GameProfile) -> Unit
    ) : RecyclerView.Adapter<ProfileAdapter.ViewHolder>() {
        
        class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
            val profileNameText: TextView = view.findViewById(R.id.profileNameText)
            val appNameText: TextView = view.findViewById(R.id.appNameText)
            val selectButton: Button = view.findViewById(R.id.selectButton)
            val editButton: ImageButton = view.findViewById(R.id.editButton)
            val deleteButton: ImageButton = view.findViewById(R.id.deleteButton)
            val assignAppButton: Button = view.findViewById(R.id.assignAppButton)
        }
        
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_profile, parent, false)
            return ViewHolder(view)
        }
        
        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val profile = profiles[position]
            val context = holder.itemView.context
            
            holder.profileNameText.text = profile.name
            
            // Получаем имя приложения по packageName
            if (profile.packageName.isNotEmpty()) {
                try {
                    val pm = context.packageManager
                    val appInfo = pm.getApplicationInfo(profile.packageName, 0)
                    holder.appNameText.text = pm.getApplicationLabel(appInfo)
                    holder.appNameText.visibility = View.VISIBLE
                } catch (e: PackageManager.NameNotFoundException) {
                    holder.appNameText.visibility = View.GONE
                }
            } else {
                holder.appNameText.visibility = View.GONE
            }
            
            holder.selectButton.setOnClickListener { onProfileSelected(profile) }
            holder.editButton.setOnClickListener { onProfileEdit(profile) }
            holder.deleteButton.setOnClickListener { onProfileDelete(profile) }
            holder.assignAppButton.setOnClickListener { onAssignApp(profile) }
        }
        
        override fun getItemCount() = profiles.size
        
        fun addProfile(profile: GameProfile) {
            profiles.add(profile)
            notifyItemInserted(profiles.size - 1)
        }
        
        fun removeProfile(profile: GameProfile) {
            val index = profiles.indexOfFirst { it.id == profile.id }
            if (index >= 0) {
                profiles.removeAt(index)
                notifyItemRemoved(index)
            }
        }
    }
}
5. /app/src/main/java/com/example/gamemapper/GestureConfigActivity.kt
kotlin

Copy
package




Продолжай

5. /app/src/main/java/com/example/gamemapper/GestureConfigActivity.kt (продолжение)
kotlin

Copy
package com.example.gamemapper

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.AdapterView
import android.widget.ArrayAdapter
import android.widget.Button
import android.widget.EditText
import android.widget.Spinner
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.gamemapper.di.Modules

class GestureConfigActivity : AppCompatActivity() {
    private lateinit var gesturesRecyclerView: RecyclerView
    private lateinit var addGestureButton: Button
    private lateinit var adapter: GestureAdapter
    private lateinit var currentProfile: GameProfile
    
    // Используем модули DI
    private val profileUseCase = Modules.profileUseCase
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_gesture_config)
        
        gesturesRecyclerView = findViewById(R.id.gesturesRecyclerView)
        addGestureButton = findViewById(R.id.addGestureButton)
        
        // Получаем текущий профиль через Use Case
        val profileId = intent.getStringExtra("profile_id") ?: return finish()
        currentProfile = profileUseCase.getProfileById(profileId) ?: return finish()
        
        title = getString(R.string.gesture_config_for, currentProfile.name)
        
        adapter = GestureAdapter(
            currentProfile.gestureMappings,
            onGestureEdit = { gesture ->
                showGestureDialog(gesture)
            },
            onGestureDelete = { gesture ->
                currentProfile.gestureMappings.remove(gesture)
                profileUseCase.saveProfile(currentProfile)
                adapter.notifyDataSetChanged()
            }
        )
        
        gesturesRecyclerView.adapter = adapter
        gesturesRecyclerView.layoutManager = LinearLayoutManager(this)
        
        addGestureButton.setOnClickListener {
            // Создаем новый жест по умолчанию
            val newGesture = GestureMapping(
                gestureType = GestureType.TAP,
                keyCode = android.view.KeyEvent.KEYCODE_UNKNOWN,
                startX = 500f,
                startY = 500f
            )
            showGestureDialog(newGesture, isNew = true)
        }
    }
    
    private fun showGestureDialog(gesture: GestureMapping, isNew: Boolean = false) {
        val dialogView = layoutInflater.inflate(R.layout.dialog_gesture_config, null)
        
        val gestureTypeSpinner = dialogView.findViewById<Spinner>(R.id.gestureTypeSpinner)
        val keyCodeSpinner = dialogView.findViewById<Spinner>(R.id.keyCodeSpinner)
        val startXEdit = dialogView.findViewById<EditText>(R.id.startXEdit)
        val startYEdit = dialogView.findViewById<EditText>(R.id.startYEdit)
        val endXEdit = dialogView.findViewById<EditText>(R.id.endXEdit)
        val endYEdit = dialogView.findViewById<EditText>(R.id.endYEdit)
        val durationEdit = dialogView.findViewById<EditText>(R.id.durationEdit)
        val tapCountEdit = dialogView.findViewById<EditText>(R.id.tapCountEdit)
        val radiusEdit = dialogView.findViewById<EditText>(R.id.radiusEdit)
        
        val endCoordinatesLayout = dialogView.findViewById<View>(R.id.endCoordinatesLayout)
        val durationLayout = dialogView.findViewById<View>(R.id.durationLayout)
        val tapCountLayout = dialogView.findViewById<View>(R.id.tapCountLayout)
        val radiusLayout = dialogView.findViewById<View>(R.id.radiusLayout)
        
        // Настраиваем спиннер типов жестов
        val gestureTypes = GestureType.values().map { it.name }
        val gestureAdapter = ArrayAdapter(this, android.R.layout.simple_spinner_item, gestureTypes)
        gestureAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
        gestureTypeSpinner.adapter = gestureAdapter
        gestureTypeSpinner.setSelection(gestureTypes.indexOf(gesture.gestureType.name))
        
        // Настраиваем спиннер кодов клавиш
        val keyCodes = arrayOf(
            android.view.KeyEvent.KEYCODE_A,
            android.view.KeyEvent.KEYCODE_B,
            android.view.KeyEvent.KEYCODE_C,
            // Добавьте больше кодов клавиш по необходимости
            android.view.KeyEvent.KEYCODE_BUTTON_A,
            android.view.KeyEvent.KEYCODE_BUTTON_B,
            android.view.KeyEvent.KEYCODE_BUTTON_X,
            android.view.KeyEvent.KEYCODE_BUTTON_Y
        )
        
        val keyCodeNames = keyCodes.map { 
            android.view.KeyEvent.keyCodeToString(it).replace("KEYCODE_", "")
        }
        
        val keyCodeAdapter = ArrayAdapter(this, android.R.layout.simple_spinner_item, keyCodeNames)
        keyCodeAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
        keyCodeSpinner.adapter = keyCodeAdapter
        
        val keyCodeIndex = keyCodes.indexOf(gesture.keyCode)
        if (keyCodeIndex >= 0) {
            keyCodeSpinner.setSelection(keyCodeIndex)
        }
        
        // Заполняем поля текущими значениями
        startXEdit.setText(gesture.startX.toString())
        startYEdit.setText(gesture.startY.toString())
        gesture.endX?.let { endXEdit.setText(it.toString()) }
        gesture.endY?.let { endYEdit.setText(it.toString()) }
        durationEdit.setText(gesture.duration.toString())
        tapCountEdit.setText(gesture.tapCount.toString())
        radiusEdit.setText(gesture.radius.toString())
        
        // Настраиваем видимость полей в зависимости от типа жеста
        gestureTypeSpinner.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
            override fun onItemSelected(parent: AdapterView<*>, view: View?, position: Int, id: Long) {
                when (GestureType.valueOf(gestureTypes[position])) {
                    GestureType.TAP -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.GONE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.LONG_PRESS -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.SWIPE -> {
                        endCoordinatesLayout.visibility = View.VISIBLE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.MULTI_TAP -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.GONE
                        tapCountLayout.visibility = View.VISIBLE
                        radiusLayout.visibility = View.GONE
                    }
                    GestureType.CIRCLE -> {
                        endCoordinatesLayout.visibility = View.GONE
                        durationLayout.visibility = View.VISIBLE
                        tapCountLayout.visibility = View.GONE
                        radiusLayout.visibility = View.VISIBLE
                    }
                }
            }
            
            override fun onNothingSelected(parent: AdapterView<*>) {}
        }
        
        // Вызываем onItemSelected вручную для установки начального состояния
        gestureTypeSpinner.onItemSelectedListener.onItemSelected(
            gestureTypeSpinner, null, gestureTypeSpinner.selectedItemPosition, 0
        )
        
        AlertDialog.Builder(this)
            .setTitle(if (isNew) R.string.add_gesture else R.string.edit_gesture)
            .setView(dialogView)
            .setPositiveButton(R.string.save) { _, _ ->
                try {
                    // Получаем значения из полей
                    val gestureType = GestureType.valueOf(gestureTypes[gestureTypeSpinner.selectedItemPosition])
                    val keyCode = keyCodes[keyCodeSpinner.selectedItemPosition]
                    val startX = startXEdit.text.toString().toFloatOrNull() ?: 0f
                    val startY = startYEdit.text.toString().toFloatOrNull() ?: 0f
                    
                    // Опциональные поля в зависимости от типа жеста
                    val endX = if (endCoordinatesLayout.visibility == View.VISIBLE)
                         endXEdit.text.toString().toFloatOrNull() else null
                    val endY = if (endCoordinatesLayout.visibility == View.VISIBLE)
                         endYEdit.text.toString().toFloatOrNull() else null
                    val duration = if (durationLayout.visibility == View.VISIBLE)
                         durationEdit.text.toString().toLongOrNull() ?: 500L else 0L
                    val tapCount = if (tapCountLayout.visibility == View.VISIBLE)
                         tapCountEdit.text.toString().toIntOrNull() ?: 1 else 1
                    val radius = if (radiusLayout.visibility == View.VISIBLE)
                         radiusEdit.text.toString().toFloatOrNull() ?: 100f else 0f
                    
                    // Создаем обновленный жест
                    val updatedGesture = GestureMapping(
                        id = gesture.id,
                        gestureType = gestureType,
                        keyCode = keyCode,
                        startX = startX,
                        startY = startY,
                        endX = endX,
                        endY = endY,
                        duration = duration,
                        tapCount = tapCount,
                        radius = radius
                    )
                    
                    // Добавляем или обновляем жест в профиле
                    if (isNew) {
                        currentProfile.gestureMappings.add(updatedGesture)
                    } else {
                        val index = currentProfile.gestureMappings.indexOfFirst { it.id == gesture.id }
                        if (index >= 0) {
                            currentProfile.gestureMappings[index] = updatedGesture
                        }
                    }
                    
                    // Сохраняем профиль через Use Case
                    profileUseCase.saveProfile(currentProfile)
                    adapter.notifyDataSetChanged()
                    
                    Toast.makeText(
                        this, 
                        if (isNew) R.string.gesture_added else R.string.gesture_updated,
                        Toast.LENGTH_SHORT
                    ).show()
                } catch (e: Exception) {
                    Toast.makeText(this, R.string.gesture_config_error, Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    class GestureAdapter(
        val gestures: MutableList<GestureMapping>,
        private val onGestureEdit: (GestureMapping) -> Unit,
        private val onGestureDelete: (GestureMapping) -> Unit
    ) : RecyclerView.Adapter<GestureAdapter.ViewHolder>() {
        
        class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
            val gestureTypeText: TextView = view.findViewById(R.id.gestureTypeText)
            val keyCodeText: TextView = view.findViewById(R.id.keyCodeText)
            val gestureDetailsText: TextView = view.findViewById(R.id.gestureDetailsText)
            val editButton: Button = view.findViewById(R.id.editButton)
            val deleteButton: Button = view.findViewById(R.id.deleteButton)
        }
        
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_gesture, parent, false)
            return ViewHolder(view)
        }
        
        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val gesture = gestures[position]
            val context = holder.itemView.context
            
            holder.gestureTypeText.text = gesture.gestureType.name
            holder.keyCodeText.text = android.view.KeyEvent.keyCodeToString(gesture.keyCode).replace("KEYCODE_", "")
            
            // Формируем детальное описание жеста в зависимости от его типа
            val details = StringBuilder()
            details.append("X: ${gesture.startX}, Y: ${gesture.startY}")
            
            when (gesture.gestureType) {
                GestureType.SWIPE -> {
                    gesture.endX?.let { endX ->
                        gesture.endY?.let { endY ->
                            details.append(" → X: $endX, Y: $endY")
                        }
                    }
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                GestureType.LONG_PRESS -> {
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                GestureType.MULTI_TAP -> {
                    details.append(", ${context.getString(R.string.tap_count)}: ${gesture.tapCount}")
                }
                GestureType.CIRCLE -> {
                    details.append(", ${context.getString(R.string.radius)}: ${gesture.radius}")
                    details.append(", ${context.getString(R.string.duration)}: ${gesture.duration}ms")
                }
                else -> {}
            }
            
            holder.gestureDetailsText.text = details.toString()
            
            holder.editButton.setOnClickListener {
                onGestureEdit(gesture)
            }
            
            holder.deleteButton.setOnClickListener {
                onGestureDelete(gesture)
            }
        }
        
        override fun getItemCount() = gestures.size
    }
}
6. /app/src/main/AndroidManifest.xml
xml

Copy
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.gamemapper">
    
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    
    <application
        android:name=".GameMapperApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GameMapper">




Продолжай

6. /app/src/main/AndroidManifest.xml (продолжение)
xml

Copy
<activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name=".EditMappingActivity"
            android:label="@string/edit_mapping"
            android:exported="false" />
        
        <activity
            android:name=".GamepadConfigActivity"
            android:label="@string/gamepad_configuration"
            android:exported="false" />
        
        <activity
            android:name=".ProfileManagementActivity"
            android:label="@string/profile_management"
            android:exported="false" />
        
        <activity
            android:name=".GestureConfigActivity"
            android:label="@string/gesture_configuration"
            android:exported="false" />
        
        <service
            android:name=".MappingService"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
            android:exported="true">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
        </service>
    </application>
</manifest>
7. /app/src/main/res/values/strings.xml (добавление новых строк)
xml

Copy
<resources>
    <!-- Существующие строки -->
    
    <!-- Новые строки для чистой архитектуры -->
    <string name="no_profiles_available">Нет доступных профилей</string>
    <string name="select_profile_for_gestures">Выберите профиль для настройки жестов</string>
    <string name="gesture_configuration">Настройка жестов</string>
    <string name="playback_completed">Воспроизведение завершено</string>
    <string name="profile_activated">Профиль \"%s\" активирован</string>
    <string name="profile_created">Профиль \"%s\" создан</string>
    <string name="profile_updated">Профиль \"%s\" обновлен</string>
    <string name="profile_deleted">Профиль \"%s\" удален</string>
    <string name="profile_name_required">Имя профиля обязательно</string>
    <string name="delete_profile_confirmation">Вы уверены, что хотите удалить профиль \"%s\"?</string>
    <string name="app_assigned">Приложение \"%1$s\" назначено профилю \"%2$s\"</string>
    <string name="gesture_added">Жест добавлен</string>
    <string name="gesture_updated">Жест обновлен</string>
    <string name="gesture_config_error">Ошибка при настройке жеста</string>
</resources>
Файлы, которые не требуют изменений
ButtonSettingsDialog.kt
CustomOverlayButton.kt
EditMappingActivity.kt
GamepadConfigActivity.kt
KeyMappingAdapter.kt
MappingService.kt
Все XML-файлы макетов, кроме добавления новых строк в strings.xml
Итоговая структура проекта
app/
├── src/
│   ├── main/
│   │   ├── java/com/example/gamemapper/
│   │   │   ├── di/
│   │   │   │   └── Modules.kt (новый)
│   │   │   ├── repository/
│   │   │   │   ├── ProfileRepository.kt (новый)
│   │   │   │   └── MappingRepository.kt (новый)
│   │   │   ├── usecase/
│   │   │   │   ├── ProfileUseCase.kt (новый)
│   │   │   │   └── MappingUseCase.kt (новый)
│   │   │   ├── ButtonSettingsDialog.kt
│   │   │   ├── CustomOverlayButton.kt
│   │   │   ├── EditMappingActivity.kt
│   │   │   ├── GameMapperApplication.kt (новый)
│   │   │   ├── GamepadConfigActivity.kt
│   │   │   ├── GameProfile.kt (изменен)
│   │   │   ├── GestureConfigActivity.kt (изменен)
│   │   │   ├── KeyMappingAdapter.kt
│   │   │   ├── MainActivity.kt (изменен)
│   │   │   ├── MappingService.kt
│   │   │   ├── PersistenceManager.kt
│   │   │   ├── ProfileManager.kt (новый)
│   │   │   ├── ProfileManagementActivity.kt (изменен)
│   │   │   ├── RecorderManager.kt (изменен)
│   │   │   └── RecorderManagerImpl.kt (новый)
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   └── (все макеты без изменений)
│   │   │   └── values/
│   │   │       └── strings.xml (добавлены новые строки)
│   │   └── AndroidManifest.xml (изменен)
│   └── test/
│       └── java/com/example/gamemapper/
│           ├── CustomOverlayButtonTest.kt (новый)
│           ├── GestureMappingTest.kt (новый)
│           ├── MappingServiceTest.kt
│           ├── PersistenceManagerTest.kt (новый)
│           ├── ProfileManagerTest.kt (новый)
│           └── RecorderManagerTest.kt
Эти изменения значительно улучшили структуру проекта, применив принципы чистой архитектуры, добавив тесты для повышения надежности и вынеся ProfileManager в отдельный файл для лучшей организации кода.