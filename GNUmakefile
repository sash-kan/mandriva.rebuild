LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
v = 0

archs = i586 x86_64
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
chroottardir = chroottars
reportdir = report
failedlogdir = failedlogs
chroottarsarchdir = $(foreach a,$(archs),$(chroottardir)/$(a))
packagesintmpinchroottar = $(foreach p,$(srpms),$(foreach d,$(chroottarsarchdir),$(d)/tmp/$(p)))
chroottars = $(foreach a,$(archs),$(chroottardir)/$(a).tar)
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(archs),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

repos1 = main contrib non-free
repos2 = release updates
repos3 = $(foreach r1,$(repos1),$(foreach r2,$(repos2),$(r1)/$(r2)))
repos  = $(foreach r,$(repos3),$(foreach p,SRPMS $(archat)/media,$(p)/$(r)))

percent = %
archat = $(strip $(foreach a,$(archs),$(findstring $(a),$@)))
ifeq ($(v),0)
	output = &>/dev/null
else
	output =
endif

.SECONDEXPANSION:

all: srpms.list $(chroottardir) $(cachedir) $(reportdir) $(chroottars) $(srpmsincache) $(reportpackages)

$(reportpackages):
	# $@ p=$(basename $(notdir $@))
	# umount sys+proc
	[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys && rm .sysmounted; :
	[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc && rm .procmounted; :
	# remove chroot
	-sudo rm -rf $(chrootdir)
	# untar chroot
	mkdir $(chrootdir)
	sudo tar -xf $(chroottardir)/$(archat).tar -C $(chrootdir)
	# install buildreqs
	sudo urpmi --noclean --no-suggests --excludedocs --no-verify-rpm --auto \
	--root $(chrootdir) --urpmi-root $(chrootdir) --buildrequires \
	$(cachedir)/$(basename $(notdir $@)) $(output)
	# install file
	# mount sys+proc
	[ ! -f .procmounted ] && sudo mount -o bind /proc $(chrootdir)/proc && touch .procmounted
	[ ! -f .sysmounted ] && sudo mount -o bind /sys $(chrootdir)/sys && touch .sysmounted
	# rpmbuild
	# check if srpm was built
	# if failed, keep log
	# write repott

# prereq = chroottars/i586/tmp packagesintmpinchroottar
#$(srpmsincache): $(chroottardir)/$(word 1,$(archs)) $$(chroottardir)/$$(word 1,$$(archs))/tmp/$$(notdir $$@)
$(srpmsincache): $$(chroottardir)/$$(word 1,$$(archs))/tmp/$$(notdir $$@)
	cp $$(ls $(chroottardir)/*/tmp/$(notdir $@)) $@

# get path to src.rpm and copy it into chroottars/i586/tmp
# if path is empty (src.rpm does not exists) then create files
# report/name.of.src.rpm.<arch> containing error message
$(packagesintmpinchroottar):
	p=$$(urpmq --urpmi-root $(chroottardir)/$(archat) \
	--sources $(basename $(notdir $@)) 2>.urpmq.error); \
	if [ -z "$$p" ] ; \
	then sudo touch $(chroottardir)/$(archat)/tmp/$(notdir $@); \
	for i in $(archs); do cat .urpmq.error > $(reportdir)/$(notdir $@).$$i; done; \
	else sudo cp $$p $(chroottardir)/$(archat)/tmp/; fi
	-rm .urpmq.error

$(cachedir) $(reportdir) $(chroottardir):
	mkdir -p $@

clearall: delete.cachedir delete.reportdir delete.chroottardir delete.chrootdir delete.failedlogdir 

delete.%:
	-sudo rm -rf $($*)

delete.chrootdir:
	-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; :
	-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; :
	-rm .sysmounted .procmounted
	-sudo rm -rf $(chrootdir)

$(chroottarsarchdir): 
	for i in $(repos); do \
	sudo urpmi.addmedia --urpmi-root $@ $$(echo $$i | sed 's!/!_!g') \
	$(mirror)/$$i; done $(output)
	sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto --root $@ \
	--urpmi-root $@ shadow-utils rpm tar basesystem-minimal rpm-build \
	rpm-mandriva-setup urpmi rsync bzip2 shadow-utils locales-en $(output)

$(chroottars): $(chroottarsarchdir)
	sudo tar -cf $@ -C $(foreach d,$^,$(findstring $(d),$@)) .

