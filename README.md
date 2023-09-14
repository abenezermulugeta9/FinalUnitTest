# FinalUnitTest

@ExtendWith(MockitoExtension.class)
public class SignalsServiceTest {
    private static final Integer ID = 1;
    @Mock
    SignalsRepository signalsRepository;
    @Mock
    SignalDefinitionsRepository signalDefinitionsRepository;
    @Mock
    SignalDefinitionsMapper signalDefinitionsMapper;
    @InjectMocks
    SignalsServiceImpl signalsService;

    @Test
    @DisplayName("GetAll_ShouldReturnListOfSignalDefinitionsDto")
    public void testGetAll() {
        List<SignalDefinitionsEntity> testEntityList = getListOfSignalDefinitionsEntity();
        SignalDefinitionsDto testDto = new SignalDefinitionsDto();

        when(signalDefinitionsRepository.findAll()).thenReturn(testEntityList);
        when(signalDefinitionsMapper.toDto(any(SignalDefinitionsEntity.class))).thenReturn(testDto);
        List<SignalDefinitionsDto> responseDtoList = signalsService.getAllSignalDefinitions();

        verify(signalDefinitionsRepository).findAll();
        verify(signalDefinitionsMapper, times(testEntityList.size())).toDto(any(SignalDefinitionsEntity.class));
        for (SignalDefinitionsEntity entity : testEntityList) {
            verify(signalDefinitionsMapper).toDto(entity);
        }
        assertEquals(responseDtoList.size(), signalDefinitionsRepository.findAll().size());
        assertNotNull(responseDtoList);
        for (SignalDefinitionsDto dto : responseDtoList) {
            assertNotNull(dto);
        }
    }

    @Test
    @DisplayName("Create_ShouldReturnSignalDefinitionsDto")
    public void testCreate() {
        SignalDefinitionsDto requestDto = getSignalDefinitionsDto();
        SignalDefinitionsEntity testEntity = getSignalDefinitionsEntity();
        SignalDefinitionsDto testDto = getSignalDefinitionsDto();

        when(signalDefinitionsMapper.toEntity(any(SignalDefinitionsDto.class))).thenReturn(testEntity);
        when(signalDefinitionsMapper.toDto(any(SignalDefinitionsEntity.class))).thenReturn(testDto);
        when(signalDefinitionsRepository.saveAndFlush(any(SignalDefinitionsEntity.class))).thenReturn(testEntity);
        SignalDefinitionsDto responseDto = signalsService.createSignalDefinition(requestDto);

        verify(signalDefinitionsMapper).toEntity(requestDto);
        verify(signalDefinitionsMapper).toDto(testEntity);
        verify(signalDefinitionsRepository).saveAndFlush(testEntity);
        assertNotNull(responseDto);
        assertEquals(responseDto.getId(), requestDto.getId());
        assertEquals(responseDto.getControlManufacturer(), signalDefinitionsRepository.saveAndFlush(testEntity).getControlManufacturer());
    }

    @Test
    @DisplayName("Update_ShouldReturnSignalDefinitionsDto")
    public void testUpdate() {
        SignalDefinitionsDto requestDto = getSignalDefinitionsDto();
        SignalDefinitionsEntity testEntity = getSignalDefinitionsEntity();
        SignalDefinitionsDto testDto = getSignalDefinitionsDto();

        when(signalDefinitionsMapper.toEntity(any(SignalDefinitionsDto.class))).thenReturn(testEntity);
        when(signalDefinitionsMapper.toDto(any(SignalDefinitionsEntity.class))).thenReturn(testDto);
        when(signalDefinitionsRepository.existsById(ID)).thenReturn(true);
        when(signalDefinitionsRepository.saveAndFlush(any(SignalDefinitionsEntity.class))).thenReturn(testEntity);
        SignalDefinitionsDto responseDto = signalsService.updateSignalDefinition(ID, requestDto);

        verify(signalDefinitionsMapper).toEntity(requestDto);
        verify(signalDefinitionsMapper).toDto(testEntity);
        verify(signalDefinitionsRepository).existsById(ID);
        verify(signalDefinitionsRepository).saveAndFlush(testEntity);
        assertNotNull(responseDto);
        assertEquals(requestDto.getId(), ID);
        assertEquals(responseDto.getControlManufacturer(), requestDto.getControlManufacturer());
        assertEquals(responseDto.getId(), signalDefinitionsRepository.saveAndFlush(testEntity).getId());
    }

    @Test
    @DisplayName("Update_ShouldThrowRuntimeException")
    public void testUpdateThrowingRuntimeException() {
        SignalDefinitionsDto requestDto = getSignalDefinitionsDtoWithInvalidId();

        Exception exception = assertThrows(RuntimeException.class, () -> signalsService.updateSignalDefinition(ID, requestDto));
        String expectedMessage = "Signal definition id does not match path id.";
        String actualMessage = exception.getMessage();

        verify(signalDefinitionsRepository, never()).saveAndFlush(any());
        verify(signalDefinitionsRepository, never()).existsById(ID);
        assertEquals(expectedMessage, actualMessage);
        assertNotEquals(requestDto.getId(), ID);
    }

    @Test
    @DisplayName("Update_ShouldThrowEntityNotFoundException")
    public void testUpdateThrowingEntityNotFoundException() {
        SignalDefinitionsDto requestDto = getSignalDefinitionsDto();

        when(signalDefinitionsRepository.existsById(ID)).thenReturn(false);
        Exception exception = assertThrows(EntityNotFoundException.class, () -> signalsService.updateSignalDefinition(ID, requestDto));
        String expectedMessage = "Signal definition not found.";
        String actualMessage = exception.getMessage();

        verify(signalDefinitionsRepository).existsById(ID);
        verify(signalDefinitionsRepository, never()).saveAndFlush(any());
        assertEquals(requestDto.getId(), ID);
        assertEquals(expectedMessage, actualMessage);
    }

    @Test
    @DisplayName("Delete_ShouldReturnVoid")
    public void testDelete() {
        SignalDefinitionsEntity testEntity = getSignalDefinitionsEntity();

        when(signalDefinitionsRepository.findById(ID)).thenReturn(Optional.of(testEntity));
        when(signalsRepository.existsBySignalDefinitionId(testEntity.getId())).thenReturn(false);

        assertAll(() -> signalsService.deleteSignalDefinitions(ID));
        verify(signalDefinitionsRepository).findById(ID);
        verify(signalDefinitionsRepository).delete(testEntity);
        verify(signalsRepository).existsBySignalDefinitionId(testEntity.getId());
    }

    @Test
    @DisplayName("Delete_ShouldThrowEntityNotFoundException")
    public void testDeleteThrowingEntityNotFoundException() {
        when(signalDefinitionsRepository.findById(ID)).thenReturn(Optional.empty());

        Exception exception = assertThrows(EntityNotFoundException.class, () -> signalsService.deleteSignalDefinitions(ID));
        String expectedMessage = "Signal definition not found!";
        String actualMessage = exception.getMessage();

        verify(signalDefinitionsRepository).findById(ID);
        verify(signalDefinitionsRepository, never()).delete(any());
        verify(signalsRepository, never()).existsBySignalDefinitionId(any());
        assertEquals(expectedMessage, actualMessage);
    }

    @Test
    @DisplayName("Delete_ShouldThrowRuntimeException")
    public void testDeleteThrowingRuntimeException() {
        SignalDefinitionsEntity testEntity = getSignalDefinitionsEntity();

        when(signalDefinitionsRepository.findById(ID)).thenReturn(Optional.of(testEntity));
        when(signalsRepository.existsBySignalDefinitionId(testEntity.getId())).thenReturn(true);

        Exception exception = assertThrows(RuntimeException.class, () -> signalsService.deleteSignalDefinitions(ID));
        String expectedMessage = "Signal definition cannot be deleted as it is currently in use!";
        String actualMessage = exception.getMessage();

        verify(signalsRepository).existsBySignalDefinitionId(testEntity.getId());
        verify(signalDefinitionsRepository).findById(ID);
        verify(signalDefinitionsRepository, never()).delete(any());
        assertEquals(expectedMessage, actualMessage);
    }

    private List<SignalDefinitionsEntity> getListOfSignalDefinitionsEntity() {
        SignalDefinitionsEntity entity1 = new SignalDefinitionsEntity();
        entity1.setId(1);
        entity1.setControlManufacturer("Fanuc");
        entity1.setCritical(true);
        entity1.setExportedKepware(false);
        entity1.setScored(true);
        entity1.setNaScore(true);
        entity1.setExportedSystem(false);
        entity1.setStdProcessVariable(true);

        SignalDefinitionsEntity entity2 = new SignalDefinitionsEntity();
        entity2.setId(2);
        entity2.setControlManufacturer("Siemens");
        entity2.setCritical(true);
        entity2.setExportedKepware(false);
        entity2.setScored(true);
        entity2.setNaScore(true);
        entity2.setExportedSystem(false);
        entity2.setStdProcessVariable(true);

        return List.of(entity1, entity2);
    }

    private SignalDefinitionsDto getSignalDefinitionsDto() {
        SignalDefinitionsDto dto = new SignalDefinitionsDto();
        dto.setId(1);
        dto.setControlManufacturer("Fanuc");
        dto.setCritical(true);
        dto.setExportedKepware(false);
        dto.setScored(true);
        dto.setNaScore(true);
        dto.setExportedSystem(false);
        dto.setStdProcessVariable(true);

        return dto;
    }

    private SignalDefinitionsDto getSignalDefinitionsDtoWithInvalidId() {
        SignalDefinitionsDto dto = new SignalDefinitionsDto();
        /** id 2 is considered invalid, because we are trying to update an entity with id 1. */
        dto.setId(2);
        dto.setControlManufacturer("Fanuc");
        dto.setCritical(true);
        dto.setExportedKepware(false);
        dto.setScored(true);
        dto.setNaScore(true);
        dto.setExportedSystem(false);
        dto.setStdProcessVariable(true);

        return dto;
    }

    private SignalDefinitionsEntity getSignalDefinitionsEntity() {
        SignalDefinitionsEntity entity = new SignalDefinitionsEntity();
        entity.setId(1);
        entity.setControlManufacturer("Fanuc");
        entity.setCritical(true);
        entity.setExportedKepware(false);
        entity.setScored(true);
        entity.setNaScore(true);
        entity.setExportedSystem(false);
        entity.setStdProcessVariable(true);

        return entity;
    }
}
